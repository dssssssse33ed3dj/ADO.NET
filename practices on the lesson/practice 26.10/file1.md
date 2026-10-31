# Практика 26.10. - за Асинхронная работа Npgsql 
- Сперва через команду ниже я запустил БД с нужным паролем (112) и портом (5432): 

<img width="1355" height="730" alt="1" src="https://github.com/user-attachments/assets/01e5d671-13a2-4443-8d92-1c8d25e3f140" />

- Cоздал папку проекта и перешел в нее: 
<img width="767" height="255" alt="2" src="https://github.com/user-attachments/assets/10d1e982-02ca-4c84-bf62-83247cdeafae" />

- Тут мы подняли изолированную базу PostgreSQL в Docker, автоматически создали и наполнили данными все таблицы через C#
- <img width="1611" height="1327" alt="3" src="https://github.com/user-attachments/assets/a4011717-e0f6-43b9-901e-e510bdb1e763" />

### Быстрый старт
- Запуск через dotnett run
- <img width="442" height="71" alt="23232323232323232323232323" src="https://github.com/user-attachments/assets/3f065470-c4c3-419b-a845-2f2ab8f637b1" />


# Задачи
# 10 комплексных задач для отработки асинхронной работы с Npgsql

### Структура проекта 
- <img width="180" height="372" alt="цууццу" src="https://github.com/user-attachments/assets/7df05a50-c73c-4979-be88-bfee9725df91" />

## Задача 1. Базовый асинхронный репозиторий с отменой и таймаутом

**Суть задачи:** реализовать DAL-метод для постраничного получения списка сотрудников с поддержкой пользовательской отмены и внутреннего таймаута; научиться различать три типа ошибок - клиентскую отмену, серверную отмену (SqlState 57014) и клиентский socket timeout.

**Что было сделано:** метод **GetEmployeesAsync(page, pageSize, userToken)** с **CancellationTokenSource.CreateLinkedTokenSource** + **CancelAfter(15s)**, **CommandTimeout = 10**, параметризованным запросом и **await using** для connection/command/reader; **OpenAsync**/**ExecuteReaderAsync**/**ReadAsync** с токеном, **.ConfigureAwait(false)** в библиотечном коде, синхронные геттеры после **ReadAsync**. В **catch** - отдельные ветки на **OperationCanceledException**, **PostgresException** с **SqlState == "57014"** и **NpgsqlException** с таймаутом. В комментариях объяснено, почему **CommandTimeout** в Npgsql - это клиентский socket timeout, закрывающий соединение, а серверный **statement_timeout** - нет. Результат: на странице 1 выведено 10 сотрудников.


- <img width="828" height="1322" alt="Первое аааа" src="https://github.com/user-attachments/assets/31e43a8e-b82f-4db3-be59-8eb82abbbdb4" />

---

## Задача 2. Массовая вставка через COPY (бинарный импорт) с прогрессом

**Суть задачи:** загрузить 1 000 000 строк в staging-таблицу через бинарный COPY (он быстрее INSERT), отслеживать прогресс через **IProgress<int>** и корректно обрабатывать отмену, чтобы таблица оставалась в исходном состоянии.

**Что было сделано:** метод **BulkCopyAsync(IEnumerable<Order>, IProgress<int>, token)** с **BeginBinaryImportAsync("COPY ... FROM STDIN (FORMAT BINARY)")**, **StartRowAsync** / **WriteAsync** с явным **NpgsqlDbType** / **CompleteAsync**; прогресс через **progress.Report** каждые 10 000 строк. При отмене **importer** закрывается без **CompleteAsync**, поэтому **DisposeAsync** откатывает операцию. В комментариях объяснено, почему COPY быстрее INSERT и почему нельзя вызывать **CompleteAsync** после исключения. Результат: 1 000 000 строк вставлено за 941 мс; при отмене на середине в таблице осталось 0 строк.

- <img width="537" height="1332" alt="второе" src="https://github.com/user-attachments/assets/ca11c9ad-5eee-455c-9c82-2906c0fbd470" />

---

## Задача 3. Транзакция с корректным откатом при отмене

**Суть задачи:** выполнить перевод средств между счетами в транзакции с уровнем изоляции Serializable и **SELECT ... FOR UPDATE**, корректно откатывая её при отмене или ошибке (в PostgreSQL после ошибки транзакция переходит в состояние «aborted» и требует явного **ROLLBACK**).

**Что было сделано:** метод **TransferAsync(fromId, toId, amount, userToken)** с **BeginTransactionAsync(IsolationLevel.Serializable)**, **LockRowAsync** с **SELECT ... FOR UPDATE** (обе строки блокируются в стабильном порядке от deadlock), двумя **UPDATE** и **CommitAsync**. В **catch** - **RollbackAsync(CancellationToken.None)**, при падении rollback - **AggregateException**. В комментариях объяснено, почему в PostgreSQL после **PostgresException** внутри транзакции все последующие команды игнорируются до **ROLLBACK**. Результат: перевод 1000 с Alice на Bob прошёл (9000 / 6000). В тесте отмена через **CancelAfter(50ms)** не успела сработать - транзакция завершилась быстрее, поэтому второй перевод (500) тоже прошёл (9500 / 5500). Механизм отката в коде реализован и работает при реальной отмене/ошибке.

- <img width="641" height="1306" alt="ТРЕТЬЕ" src="https://github.com/user-attachments/assets/4702f59c-f1d2-407b-8170-4a131afe0ce1" />

---

## Задача 4. SequentialAccess + потоковая обработка bytea (BLOB)

**Суть задачи:** выгрузить документы из колонки **bytea** в файловую систему, читая их в режиме **SequentialAccess**, чтобы не грузить в память сразу всё; понять ограничения **bytea** для очень больших BLOB.

**Что было сделано:** метод **ExportDocumentsAsync(outputDir, token)** с **ExecuteReaderAsync(CommandBehavior.SequentialAccess)**, синхронным чтением **id** и **file_name**, **GetFieldValueAsync<byte[]>(2, token)** для **content** и записью в **FileStream** с **FileOptions.Asynchronous**. В комментариях отмечено: при нарушении порядка чтения колонок в sequential-режиме будет ошибка; **GetFieldValueAsync<byte[]>** всё равно загружает весь bytea в массив, поэтому для больших BLOB альтернатива - Large Objects (**lo_read**/**lo_write**) или хранение файлов вне БД. Результат: 5 файлов (**01_doc_1.bin** … **05_doc_5.bin**, 1024–5120 байт) выгружены и совпадают по размеру с содержимым БД.

- <img width="893" height="1258" alt="читвертая" src="https://github.com/user-attachments/assets/0063651e-dcd3-43b8-b9f8-30047f61eb44" />

---

## Задача 5. Батчинг вместо MARS: несколько запросов за один round-trip

**Суть задачи:** выполнить несколько независимых запросов к БД за один round-trip через **NpgsqlBatch** (в Npgsql нет MARS, как в SQL Server), понять, что батч экономит сетевую задержку, но не даёт параллелизма на сервере.

**Что было сделано:** метод **BuildReportAsync(customerId, token)** с **NpgsqlBatch** из трёх команд (заказы клиента, позиции через JOIN, сумма), чтением результатов через **ExecuteReaderAsync** и **NextResultAsync**. Отдельный метод **CompareAsync** замеряет три подхода: один **NpgsqlBatch**, три запроса на одном соединении последовательно, три запроса на трёх соединениях параллельно. В комментариях объяснено, что batch ≠ MARS и что для настоящего параллелизма нужны отдельные соединения из пула. Результат: Batch - 1 мс, последовательно на 1 conn - 4 мс, параллельно на 3 conn - 9 мс (на localhost разница мала, параллельный вариант дороже за счёт открытия соединений).

- <img width="762" height="1315" alt="пятаёе" src="https://github.com/user-attachments/assets/8db68cb9-3624-4da8-9ea4-13ecbb27c16a" />

---

## Задача 6. Пул соединений Npgsql: исчерпание и диагностика

**Суть задачи:** воспроизвести ошибку «The connection pool has been exhausted», понять разницу между клиентским **Maximum Pool Size** и серверным **max_connections**, а также «вылечить» пул через **NpgsqlConnection.ClearAllPools**.

**Что было сделано:** строка подключения с **Maximum Pool Size=5;Timeout=5**. Метод **ExhaustPoolAsync** держит 5 соединений открытыми; **DiagnosePoolAsync** пытается открыть 6-е, замеряет время и ловит **NpgsqlException** с текстом про pool exhausted; **HealPool** освобождает соединения и вызывает **ClearAllPools**. В комментариях объяснено, что **Maximum Pool Size** (клиент) и **max_connections** (сервер) - разные лимиты, и как connection leak выглядит снаружи. Результат: 6-е соединение упало за 5013 мс с **The connection pool has been exhausted**; после **ClearAllPools** новое соединение открылось, на сервере - 1 соединение.

- <img width="954" height="1257" alt="шистое" src="https://github.com/user-attachments/assets/e4b90e90-ad97-4041-aee8-8ee87fafb8f8" />

---

## Задача 7. Health check с statement_timeout и Cancel

**Суть задачи:** сделать компонент **IHealthCheck** для ASP.NET Core, который быстро и безопасно проверяет доступность PostgreSQL, используя и клиентский **CancelAfter**, и серверный **statement_timeout**, и возвращает **Healthy** / **Degraded** / **Unhealthy** с временем в **data**.

**Что было сделано:** класс **DatabaseHealthCheck : IHealthCheck** с внутренним **CancelAfter(3s)**, **Timeout=5** в connection string, **SET statement_timeout = '2000ms'** после **OpenAsync** и **SELECT 1**; ловит **PostgresException** с **SqlState 57014** (сработал statement_timeout) и **OperationCanceledException** (сработал CancelAfter), возвращает **Healthy** / **Degraded** (если > 1000 мс) / **Unhealthy** с временем. В комментариях объяснено, почему **CommandTimeout** закрывает соединение, а **statement_timeout** - нет, и почему health check должен использовать именно **statement_timeout**. Результат: **Healthy - 148 мс**; **pg_sleep(10)** → **Unhealthy - 2016 мс** (statement_timeout, соединение живое); неверный порт → **Unhealthy - Failed to connect to 127.0.0.1:5999**.

- <img width="947" height="1258" alt="семь" src="https://github.com/user-attachments/assets/b7da1d01-6212-42b8-8826-cf5aedcbf613" />

---

**Итог:** я научился писать асинхронный DAL, корректно работающий с отменой, таймаутами, транзакциями, массовыми операциями и пулом соединений, различаещийц типы ошибок. 
