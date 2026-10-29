# Конспект за 29.10

## Проект: Мини-трекер задач (Dapper + PostgreSQL)

**Цель проекта:** Изучить Dapper в связке с PostgreSQL (массивы, JSONB, RETURNING, транзакции). Написать консольное приложение для управления проектами и задачами.

**Стек и окружение:** 
Для работы я использовал **Docker Desktop** в связке с **Visual Studio (VS Code)**. Вместо pgAdmin я решил не устанавливать отдельный клиент, так как управлять локальной базой из терминала мне показалось удобнее. Также используется порт 5433, так как стандартный 5432 был занят локальным PostgreSQL
Рабочую локальную БД и таблицы я поднял за 2 команды в терминале:


Все скриншоты с выполненными командами и результатами работы лежат по пути: 
`ADO.NET/conspect/conspect for 29.10/screenshots for 29.10/`


**Запуск контейнера с базой данных:**
   ```
   docker run --name tasktracker-db -e POSTGRES_PASSWORD=secret -e POSTGRES_DB=tasktracker -p 5433:5432 -d postgres:16
    ```
(Примечание: я использовал в строке подключения в appsettings.json порт 5433, так как стандартный 5432 был занят локальным PostgreSQL).

Создание таблиц из файла Schema.sql:

 ```
docker exec -i tasktracker-db psql -U postgres -d tasktracker < Data\Schema.sql
Проверка, что таблицы создались:

docker exec -it tasktracker-db psql -U postgres -d tasktracker -c "\dt"
 ```

Структура проекта

 ```
TaskTracker/
├── TaskTracker.csproj
├── Program.cs
├── appsettings.json
├── Data/
│   ├── IDbConnectionFactory.cs
│   ├── PostgresConnectionFactory.cs
│   └── Schema.sql
├── Models/
│   ├── Project.cs
│   ├── TaskItem.cs
│   └── User.cs
└── Repositories/
    ├── ProjectRepository.cs
    ├── TaskRepository.cs
    └── UserRepository.cs
     ```
### Что было сделано по шагам
1. Развертывание и настройка
Поднял БД в Docker, настроил строку подключения и фабрику IDbConnectionFactory. Включил MatchNamesWithUnderscores = true для автоматического маппинга snake_case из БД в PascalCase C#.

2. Базовый CRUD и RETURNING
Реализовал базовые операции через QueryAsync и ExecuteAsync. Использовал преимущество PostgreSQL - оператор RETURNING для получения сгенерированных ID и вставленных строк за один запрос.

3. Работа с массивами и JSONB
Научился передавать массивы в SQL через ANY (вместо IN). Разобрался с JSONB: настроил чтение, запись с приведением ::jsonb и обновление ключей через оператор конкатенации ||.

4. Починка багов из конспекта
Столкнулся с ошибкой DateOnly в Dapper 2.1.35. Решил обновлением пакета до 2.1.44. Также исправил баг конспекта с splitOn: "id" в связях один-ко-многим, добавив уникальные алиасы (task_id) для корректной группировки задач.

5. Массовые операции
Сравнил скорость вставки 100 000 строк. Выяснил, что бинарный COPY (BeginBinaryImportAsync) работает в разы быстрее, чем обычный INSERT через unnest.

6. Пагинация и Транзакции
Сделал обычную пагинацию через OFFSET/LIMIT (через QueryMultiple). Освоил транзакции: обернул создание проекта и его первой задачи «Kickoff» в атомарную операцию с BeginTransactionAsync.

### Выполнение заданий для самостоятельной отработки (раздел 21.13)
Сделал все 6 дополнительных заданий:

- 1. GetOverdueAsync (Просроченные задачи).
Задание: найти задачи с due_date < CURRENT_DATE и is_done = false.
Сделано: добавил метод в TaskRepository, использовал QueryAsync<TaskItem>. Проверил — находит вчерашнюю задачу.

- 2. UpdateSettingsAsync (Обновление JSONB).
Задание: обновить настройки через settings || @Patch::jsonb.
Сделано: добавил метод в ProjectRepository. Проверил через Docker — старые ключи сохраняются, новые добавляются.

- 3. GetPageKeysetAsync (Keyset-пагинация).
Задание: реализовать пагинацию по id > @LastId.
Сделано: добавил метод в ProjectRepository. Проверил — при LastId=0 выводит все проекты, при LastId=1 — только последующие.

- 4. BulkInsertCopyAsync (Замена на COPY).
Задание: заменить BulkInsertAsync на BeginBinaryImportAsync и сравнить скорость.
Сделано: добавил метод BulkInsertCopyAsync в TaskRepository. Замерил через Stopwatch на 100 000 строк — COPY оказался в разы быстрее, чем unnest.

- 5. GetManyWithTasksAsync (Несколько проектов).
Задание: получить несколько проектов с задачами за один запрос с группировкой в словарь.
Сделано: добавил метод в ProjectRepository, использовал splitOn с алиасами. Проверил — задачи распределяются по своим проектам.

- 6. CreateWithKickoffAsync (Транзакция).
Задание: обернуть создание проекта в транзакцию и добавить первую задачу «Kickoff».
Сделано: добавил метод в ProjectRepository с BeginTransactionAsync и CommitAsync. Проверил через Docker — проект и задача создаются вместе (или откатываются при ошибке).