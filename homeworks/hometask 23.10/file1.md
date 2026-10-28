# Домашняя работа от 23.10
Мой стек разработки:
- Редактор кода: Visual Studio Code
- Развертывание базы данных и окружения: Docker Desktop (PostgreSQL)
- Работа с базой: Локально через ADO.NET (Npgsql), без использования внешних GUI (pgAdmin).


---
# 10 задач по ADO.NET для проекта «Автозапчасти»
# 1. Тема задачи: Поиск запчастей по названию и артикулу
- Описание задачи: Добавь метод SearchParts(string query) в DbHelper. Он должен возвращать список запчастей, у которых name или article содержат подстроку query (регистронезависимо). Используй параметризованный ILIKE с @q, значение - параметра — $"%{query}%". Если query пустой — вернуть все запчасти. В контроллере сделай action Search, во view — строку поиска над таблицей.  
- Что было сделано мною:
открыл папку проекта autoparts-docker в VS Code. В Data/DbHelper.cs добавил метод SearchParts(query). Для защиты от SQL-инъекций применил параметризацию через @q. Поиск сделан с помощью оператора ILIKE для игнорирования регистра символов, а знаки % (подстановка $"%{query}%") нужны для поиска подстроки в любом месте полей name и article. Если входящая строка пустая, метод возвращает полный список запчастей.

- <img width="857" height="741" alt="fddfdf" src="https://github.com/user-attachments/assets/04a3b340-20dd-41af-9d0d-429e362ddf4c" />

Далее добавил в Controllers/PartsController.cs этот экшен новый. Теперь он который принимает строку из инпута, обращается к _db.SearchParts(query) и возвращает отфильтрованные данные в представление Index. Также использовал ViewBag.Query, чтобы введённый пользователем текст сохранялся в строке поиска после перезагрузки страницы.
- <img width="767" height="752" alt="dftdfdfdfdfdfdfdfdfdfdfdf" src="https://github.com/user-attachments/assets/ba7adf8b-76f7-4e24-b4a6-cff4f2b99753" />

Потом добавил в разметку Views/Parts/Index.cshtml следующий кусок. Он нужен для поиска с методом GET над таблицей. Текстовое поле связано с параметром name="query", и данные уходят в созданный экшен контроллера. Есть кнопка «Сбросить» фильтр, перенаправляя на базовый метод Index.
- Было: <img width="880" height="222" alt="ва12" src="https://github.com/user-attachments/assets/d00c7efd-93c3-46ab-8172-69bae0075fa6" />
- У меня была ошибка со стандартными HTML-формами таблицы в браузере (ничего не реагировало на сайте), и отправка запроса сделана теперь через базовый JavaScript-скрипт на событии onclick - при клике на «Найти» скрипт считывает текст из инпута и перенаправляет браузер на точный адрес /Parts/Search?query=.... 
- Стало: <img width="1047" height="237" alt="5656565656" src="https://github.com/user-attachments/assets/022e38f0-93ad-490a-9811-2b89eaa8f163" />


- На [localhost](http://localhost:8080/Parts) я потыкал вот что: нашёл Масляной фильтр и вывел его, а также сбросил через кнопку его и вывел уже все позиции. Фильтр работает успешно
- <img width="1002" height="332" alt="787878" src="https://github.com/user-attachments/assets/b181d2fb-d847-4af8-b670-c49cc159bd1e" />


# 2. Тема задачи: Удаление запчасти с проверкой результата
- Описание задачи: Добавь DeletePart(int id), который выполняет DELETE FROM parts WHERE id = @id и возвращает true, только если ExecuteNonQuery() вернул больше нуля. В контроллере — POST-action с антифорджери, во view — кнопка «Удалить» с confirm(). Если запись не найдена — показать сообщение через TempData.

- Что было сделано мною:
В Data/DbHelper.cs добавил метод DeletePart(int id). Удаление сделано с помощью SQL-оператора DELETE FROM parts WHERE id = @id. Метод вызывает ExecuteNonQuery(), который возвращает количество фактически измененных (удаленных) строк в базе данных. Логика метода построена так, что он возвращает true только в случае, если это количество строго больше нуля (строка действительно существовала и была стерта), иначе возвращает false.
- <img width="567" height="271" alt="del" src="https://github.com/user-attachments/assets/2234b564-99f7-4a38-a7c7-de0e0ce11af6" />

Далее добавил в Controllers/PartsController.cs новый POST-экшен Delete(int id). добавил атрибут защитный [ValidateAntiForgeryToken], он блокирует CSRF-атаки. Экшен вызывает созданный метод удаления _db.DeletePart(id) и записывает текстовое уведомление в объект TempData под ключами "SuccessMessage" или "ErrorMessage". 
- <img width="747" height="277" alt="67" src="https://github.com/user-attachments/assets/3a69cb18-a31e-438a-8a02-4a1e16330af1" />

  
Потом добавил изменения в разметку страницы Views/Parts/Index.cshtml. В самом верху файла прописал блоки алертов для вывода сообщений из TempData. Внутри тега таблицы добавил новую колонку «Удаление». При клике на неё сначала срабатывает стандартное окно браузера confirm(). В случае согласия пользователя JavaScript-функция  находит на странице скрытый валидный токен безопасности @Html.AntiForgeryToken(), генерирует POST-запрос и безопасно отправляет его на адрес /Parts/Delete/id.

- <img width="955" height="341" alt="dsds" src="https://github.com/user-attachments/assets/0c1251a8-3ddd-4799-bf85-598ebe18a970" />

- На localhost:
открыл страницу каталога http://localhost:8080/Parts, нашел одну из позиций и нажал кнопку «Удалить». Убедился, что корректно выводит всплывающее окно confirm с вопросом подтверждения. После «ОК» выбранная деталь навсегда пропала из списка 

- <img width="1990" height="910" alt="ВАВАВАВАВАВАВАВА" src="https://github.com/user-attachments/assets/1401bdcf-6177-444c-83d8-648d205a64a3" />
- <img width="1551" height="577" alt="0999999999999" src="https://github.com/user-attachments/assets/0976af95-03ed-4f74-aadb-61211a9080df" />





# 3. Тема задачи: Статистика по складу одним запросом

- Что было сделано мною:
В Data/DbHelper.cs добавил метод GetStats(). Метод возвращает кортеж из трех значений. Написал SQL-запрос с агрегатными функциями COUNT, SUM и AVG. Обернул расчеты в COALESCE(..., 0) для защиты от пустой базы. Использовал NpgsqlDataReader для чтения трех колонок одной строки. Метод ExecuteScalar здесь не подходит.
- <img width="838" height="352" alt="433333" src="https://github.com/user-attachments/assets/ba016381-cb19-4158-a500-ed414bc92a27" />

Далее добавил в Controllers/PartsController.cs новый экшен Stats(). Этот метод обрабатывает GET-запросы. Он вызывает метод _db.GetStats(). Полученный кортеж передается напрямую в новое представление.
- <img width="496" height="157" alt="45666666666666666666" src="https://github.com/user-attachments/assets/10972663-38a6-4bb2-bb10-97ce3c5362ab" />

Потом создал отдельный файл разметки Views/Parts/Stats.cshtml. Настроил прием модели в виде кортежа. Создал три независимые карточки с цветным фоном. Вывел значения через Razor-синтаксис. Применил формат ToString("0.00") с припиской «руб.». Это защищает от ошибок локали в Docker-контейнере Linux. Внизу добавил прямую ссылку href="/Parts" для возврата в каталог.
- Было: страницы со статистикой не существовало, выдавало ошибку 404.
- Стало: появился новый файл Stats.cshtml с тремя визуальными блоками.

- <img width="1271" height="610" alt="12121212121212" src="https://github.com/user-attachments/assets/b801b3b8-64b5-4a88-abc7-377286124c4c" />

- На localhost:
ввел в браузере прямой путь http://localhost:8080/Parts/Stats. Страница успешно загрузилась. На экране отобразились три цветные карточки с посчитанной аналитикой. Нажатием на кнопку «Назад к списку» проверил возврат в каталог.
- <img width="2581" height="843" alt="0-0000087" src="https://github.com/user-attachments/assets/d7e493d4-1911-45f8-bc7f-5c8fd66c6b29" />


# 4. Тема задачи: Оформление заказа в транзакции
- Описание задачи: Создай таблицу orders (id SERIAL, part_id INT, quantity INT, total NUMERIC, created_at TIMESTAMP). Метод PlaceOrder(int partId, int quantity) должен в одной транзакции: прочитать price и stock из parts, проверить что stock >= quantity, вставить запись в orders, выполнить UPDATE parts SET stock = stock - @qty WHERE id = @id, закоммитить. При нехватке товара — Rollback и false. Все NpgsqlCommand внутри транзакции обязаны получать объект tx.

- Что было сделано мною:
В Data/DbHelper.cs внутри метода Initialize() добавил скрипт создания таблицы orders. Таблица содержит id, part_id, quantity, total и created_at.
потом в класс DbHelper.cs добавил метод PlaceOrder(int partId, int quantity). Метод открывает соединение и создает транзакцию через connection.BeginTransaction(). Первым запросом считываю price и stock из таблицы parts. В C# коде проверяю условие stock >= quantity. Если товара мало, вызываю transaction.Rollback() и возвращаю false. При успехе вставляю запись в orders и уменьшаю остаток в parts через UPDATE. Все команды получают объект транзакции cmd.Transaction = tx. В конце вызываю transaction.Commit() для сохранения данных.

- <img width="738" height="732" alt="565656565656565656565656565656565656565656565656" src="https://github.com/user-attachments/assets/94050358-9a87-4f8b-b695-24fcd72fa79b" />

Далее добавил в Controllers/PartsController.cs новый POST-экшен Order(int id, int quantity). Метод принимает данные из формы. Он вызывает созданный метод _db.PlaceOrder(id, quantity). Результат операции записывается в TempData["Message"]. После этого контроллер перенаправляет пользователя на главную страницу каталога.
- <img width="616" height="190" alt="11111" src="https://github.com/user-attachments/assets/0870d86b-c69b-4c09-95f0-dada8d614e37" />

Потом внес изменения в разметку страницы Views/Parts/Index.cshtml. Внутри цикла foreach таблицы заменил старую форму покупки Buy на новую форму Order. Она шлет POST-запрос на адрес /Parts/Order. Передача параметров id и quantity идет через скрытые и числовые поля ввода.
- Было: форма покупки была завязана на старый метод Buy без транзакции.
- Стало: форма перенаправляет запрос на транзакционный экшен Order.
- <img width="846" height="162" alt="766666666666" src="https://github.com/user-attachments/assets/a0f55878-7ecd-4ff1-a086-4a87248d0aec" />

- На localhost:
перезапустил контейнеры. Таблица orders автоматически создалась в PostgreSQL. В каталоге у тормозной колодки ввел количество 5 и нажал кнопку покупки. Страница обновилась с сообщением об успехе, а остаток уменьшился. Затем ввел количество 500 для проверки нехватки товара. Транзакция успешно откатилась, и сайт вывел ошибку о недостатке запчастей.
- <img width="1281" height="730" alt="1" src="https://github.com/user-attachments/assets/4a1fadd0-350a-4a22-a57a-fb9e476ba12c" />
- <img width="1118" height="671" alt="2" src="https://github.com/user-attachments/assets/0c239520-84ce-483f-a474-557ffd97df4a" />



# 5. Тема задачи: Массовое поступление товара с откатом при ошибке
- Описание задачи: Метод Restock(List<(int PartId, int AddQty)> items) в одной транзакции для каждого элемента выполняет UPDATE parts SET stock = stock + @qty WHERE id = @id. Если хотя бы один PartId не найден (ExecuteNonQuery() == 0) — откатить всю транзакцию целиком. Во view — форма с несколькими строками «id + количество».

- Что было сделано мною:
 В Data/DbHelper.cs добавил метод Restock(List<(int PartId, int AddQty)> items). Метод открывает соединение и создает транзакцию. Внутри цикла foreach выполняю команду UPDATE parts SET stock = stock + @qty WHERE id = @id. Каждая команда принудительно получает объект транзакции cmd.Transaction = tx. Проверяю результат выполнения через ExecuteNonQuery(). Это значит, что деталь не найдена. Если все строки успешно обновились, вызываю tx.Commit() и возвращаю true.
- <img width="607" height="680" alt="хуйня" src="https://github.com/user-attachments/assets/800ee4de-ca46-42c0-b1a4-510bc4efbd36" />

Добавил в Controllers/PartsController.cs новый экшен Restock(string data). Метод принимает данные в виде строки через GET-запрос. В C# коде парсю эту строку на пары ID и количество. Собираю их в список кортежей List<(int PartId, int AddQty)>. Передаю список в созданный метод _db.Restock(items). Результат выполнения заношу в TempData["SuccessMessage"] или TempData["ErrorMessage"]. Контроллер перенаправляет пользователя на базовый каталог.

- <img width="890" height="802" alt="ВВВВВВВВВВВ" src="https://github.com/user-attachments/assets/64a1bace-58ff-4902-a266-931c9d6ff569" />

 добавил изменения в разметку страницы Views/Parts/Index.cshtml. В самом низу страницы под таблицей разместил новую панель пополнения склада. Она содержит три строки текстовых полей ввода для ID и количества. Кнопка «Обновить склад» считывает эти поля через JavaScript. Скрипт формирует строку данных и перенаправляет браузер на адрес /Parts/Restock?data=.... 
- Было: на сайте отсутствовала возможность массового пополнения склада.
- Стало: под таблицей появилась независимая панель массового restock-поступления на JavaScript.
- <img width="1052" height="642" alt="11111111111111111111111111111111111111111" src="https://github.com/user-attachments/assets/ef2e5086-9a61-4c63-8f77-035a79046720" />

- На localhost я потыкал вот что:
В панели массового поступления ввел в первую строку ID 1 и количество 10, во вторую строку ID 3 и количество 20. Нажал кнопку «Обновить склад». Страница обновилась с зеленым алертом, а остатки товаров на складе успешно увеличились. Для проверки отката ввел во вторую строку несуществующий ID 999. Транзакция успешно отменила все изменения для всей партии, и остатки товаров не изменились.
- <img width="1173" height="942" alt="ВАЛЬЛЬЛЬЛЬЛЬЛЬЛЬЛЬЛЬЛЬ" src="https://github.com/user-attachments/assets/d306a6da-565c-47ee-ae65-25a3d90383bf" />

- <img width="968" height="183" alt="ОШИБАП" src="https://github.com/user-attachments/assets/f744b238-0787-4b8f-bfa8-24c8ba968438" />



# 6. Тема задачи: «Сырое» чтение через NpgsqlDataReader
- Описание задачи: Метод GetRawParts() выполняет SELECT * FROM parts и вручную раскладывает каждую строку в Dictionary<string, object?>, используя reader.FieldCount, reader.GetName(i), reader.GetValue(i) и reader.IsDBNull(i). Модель Part не использовать. Во view вывести таблицу, заголовки — из ключей словаря. Цель — прочувствовать reader без обёрток.
- Что было сделано мною:
В Data/DbHelper.cs добавил метод GetRawParts(). Он возвращает список словарей List<Dictionary<string, object?>>. Метод выполняет прямой SQL-запрос SELECT * FROM parts. В цикле while (reader.Read()) использую свойство reader.FieldCount для обхода колонок. Имя каждого столбца вытаскиваю динамически через reader.GetName(i). Значение ячейки забираю с помощью reader.GetValue(i). Проверяю базу на наличие пустых значений через reader.IsDBNull(i). Модель Part внутри метода полностью проигнорирована.

- <img width="648" height="437" alt="ЛЛЛЛЛЛЛЛЛЛ" src="https://github.com/user-attachments/assets/1bd0b072-8aab-4cf6-af51-e28d67483f93" />

Далее добавил в Controllers/PartsController.cs новый экшен Raw(). Метод обрабатывает входящие GET-запросы на просмотр сырых данных базы. Он обращается к написанному методу _db.GetRawParts(). Полученную коллекцию словарей экшен передает напрямую в новое изолированное представление.

- <img width="442" height="171" alt="МАПССССССС" src="https://github.com/user-attachments/assets/749157dd-7291-47a5-882c-7600403a3ef3" />

Потом создал новый файл разметки Views/Parts/Raw.cshtml. Настроил прием коллекции словарей в качестве модели. На странице отрисовал стандартную HTML-таблицу. Шапку таблицы <th> заполнил автоматически из ключей самого первого элемента коллекции. Строки таблицы вывожу с помощью вложенных циклов foreach, перебирая все значения каждого словаря. В самом низу добавил ссылку-кнопку href="/Parts" для быстрого возврата в каталог.
- Было: страницы просмотра сырых данных не существовало, сервер возвращал ошибку 404.
- Стало: появилось представление Raw.cshtml, собирающее таблицу динамически из ключей словаря.
- <img width="1577" height="771" alt="СВЫЧ" src="https://github.com/user-attachments/assets/0e6468df-9141-4c4d-886c-9dc69e5b4203" />


- На localhost я потыкал вот что:
ввел в браузере прямой путь http://localhost:8080/Parts/Raw. Страница успешно открылась без сбоев. Таблица вывелась со всеми колонками напрямую из СУБД. Названия заголовков автоматически совпали с именами полей в PostgreSQL. Ссылка возврата в каталог сработала корректно.
- <img width="1176" height="543" alt="RAW" src="https://github.com/user-attachments/assets/780a27ba-aff3-4572-a0a1-45d6eb12f6af" />



# 7. Тема задачи: DataTable через NpgsqlDataAdapter
- Описание задачи: Метод GetPartsDataTable() заполняет DataTable через NpgsqlDataAdapter и adapter.Fill(dt). Контроллер передаёт DataTable в view, view печатает строки через foreach (DataRow row in Model.Rows). Дополнительно сделай GetPartsDataSet() с двумя таблицами (parts и orders) в одном DataSet — чтобы увидеть, зачем DataSet вообще нужен.


- Что было сделано мною:
В Data/DbHelper.cs добавил метод GetPartsDataTable(). Метод создает объект DataTable и заполняет его с помощью NpgsqlDataAdapter через метод adapter.Fill(dt). Также добавил метод GetPartsDataSet(). Он создает объект DataSet и заполняет в него сразу две таблицы: parts и orders одним махом. Это позволяет увидеть, как DataSet выступает локальным хранилищем нескольких таблиц со связями.
- <img width="692" height="325" alt="7" src="https://github.com/user-attachments/assets/50f4ed46-cac6-4a72-bcd5-7ee28d66a2f2" />

Далее добавил в Controllers/PartsController.cs новый экшен Table(). Метод обрабатывает входящие GET-запросы. Он вызывает написанный метод _db.GetPartsDataTable(). Полученный объект DataTable контроллер передает напрямую в новое изолированное представление.
- <img width="373" height="222" alt="90" src="https://github.com/user-attachments/assets/228f5888-cca7-4060-920b-d3cc80f9b85f" />

Потом создал новый файл разметки Views/Parts/Table.cshtml. Настроил прием объекта DataTable в качестве модели страницы. Отрисовал стандартную HTML-таблицу. Строки таблицы вывожу с помощью цикла foreach (DataRow row in Model.Rows). Значения ячеек забираю по строковым именам колонок. В самом низу добавил ссылку-кнопку href="/Parts" для возврата в каталог.
- Было: страницы просмотра данных через DataTable не существовало, сервер возвращал ошибку 404.
- Стало: появилось представление Table.cshtml, выводящее строки через коллекцию Model.Rows.

- <img width="1528" height="767" alt="0-0-0-0-0-0-0-0-" src="https://github.com/user-attachments/assets/f7bca2ca-6c8e-4f27-8e19-87238de2befb" />

- На localhost:
ввел в браузере прямой путь http://localhost:8080/Parts/Table. Страница успешно открылась без сбоев. Таблица на основе связного слоя ADO.NET вывелась со всеми позициями. Ссылка возврата в каталог сработала корректно.
- <img width="906" height="547" alt="0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-" src="https://github.com/user-attachments/assets/a4f03e2b-6ec2-4af6-adf1-79107b9edec9" />



# 8. Тема задачи: Асинхронная версия GetAllParts
- Описание задачи: Перепиши GetAllParts в GetAllPartsAsync(), используя OpenAsync, ExecuteReaderAsync, ReadAsync, await using. Контроллер Index должен стать async Task<IActionResult> и вызывать await _db.GetAllPartsAsync(). Убедись, что страница работает. Никаких .Result и .Wait() — только await.

- Что было сделано мною:
 В Data/DbHelper.cs добавил асинхронный метод GetAllPartsAsync(), возвращающий Task<List<Part>>. Заменил синхронное открытие соединения на await connection.OpenAsync(). Создание ридера переписал на асинхронное выполнение await using var reader = await cmd.ExecuteReaderAsync(). Чтение строк в цикле перевел на неблокирующий вызов await reader.ReadAsync(). Синхронные блокировки .Result или .Wait() полностью исключены из кода бэкенда.

- <img width="692" height="552" alt="8" src="https://github.com/user-attachments/assets/1104cc0b-6768-4b3a-8439-38cdd2ef1daa" />

Далее внес изменения в Controllers/PartsController.cs. Наш базовый экшен Index() полностью переписал с использованием ключевых слов async Task<IActionResult>. Внутри метода заменил старый синхронный вызов на await _db.GetAllPartsAsync(). Это позволяет серверу эффективно обрабатывать другие запросы, пока база данных собирает список запчастей.

- <img width="692" height="552" alt="8" src="https://github.com/user-attachments/assets/1be659f1-3664-4c2a-8a2b-2defc59607d1" />

Потом проверил разметку страницы Views/Parts/Index.cshtml. На фронтенде никаких изменений не потребовалось. Модель страницы по-прежнему принимает стандартную коллекцию List<Part>. Асинхронность полностью скрыта на уровне бэкенда и контроллера.
- Было: метод Index контроллера и метод GetAllParts в хелпере работали синхронно, блокируя потоки сервера.
- Стало: весь процесс выборки каталога переведен на асинхронные вызовы async / await.

- На localhost я потыкал вот что:
перезапустил контейнеры Docker Desktop. Открыл главную страницу каталога по адресу http://localhost:8080/Parts. Убедился, что каталог запчастей успешно загрузился и отобразился на экране. Проверил работу кнопок покупки, удаления и панели поиска. Асинхронная версия страницы работает стабильно и без сбоев.
- <img width="1032" height="853" alt="0-0-0-0-0-0-0-0-0-мсассссссссс" src="https://github.com/user-attachments/assets/4f2dfb0a-2b88-41d5-8010-abefe46e7c36" />



# 9. Тема задачи: Обработка уникального ограничения (SqlState)
- Описание задачи: Добавь в Initialize() ограничение ALTER TABLE parts ADD CONSTRAINT uq_parts_article UNIQUE (article). Метод TryAddPart(Part part) возвращает (bool Ok, string? Error): пытается вставить запись, а при PostgresException с SqlState == "23505" возвращает (false, "Запчасть с таким артикулом уже существует"). В контроллере при ошибке — вернуть форму с ModelState.AddModelError. Проверь: добавь дубль артикула, убедись что сайт показывает понятную ошибку, а не 500.

- Что было сделано мною:
В Data/DbHelper.cs внутри метода Initialize() добавил SQL-команду создания уникального индекса ALTER TABLE parts ADD CONSTRAINT uq_parts_article UNIQUE (article). Далее добавил метод TryAddPart(Part part), возвращающий кортеж (bool Ok, string? Error). Обернул команду в блок try-catch. Перехватываю исключение PostgresException. Проверяю его свойство ex.SqlState == "23505". Если код совпал, возвращаю ошибку об уже существующем артикуле. При успехе возвращаю результат (true, null).
- <img width="583" height="262" alt="ВЫСМСМ" src="https://github.com/user-attachments/assets/b4ac2b8f-2bd6-48f2-9e2c-24717c083990" />

Далее внес изменения в Controllers/PartsController.cs. Нашел существующий POST-экшен Create(Part part) для добавления деталей. Заменил в нем старый метод _db.AddPart(part) на новый вызов _db.TryAddPart(part). Если метод вернул ошибку уникальности, записываю её в форму через ModelState.AddModelError("Article", result.Error). Контроллер возвращает пользователя на ту же форму добавления с заполненными полями.

- <img width="1165" height="897" alt="СМББББББББББ" src="https://github.com/user-attachments/assets/53eba42b-04fd-458b-87f7-c11179838022" />

Потом проверил разметку страницы Views/Parts/Create.cshtml. Внутри формы добавления запчасти проверил наличие тега валидации ошибок <span asp-validation-for="Article" class="text-danger"></span>. Ошибки из ModelState автоматически выводятся в этот тег красным цветом.
- Было: при вводе дублирующегося артикула сервер выдавал критическую ошибку 500 (Internal Server Error).
- Стало: бэкенд безопасно перехватывает сбой и выводит текстовое предупреждение прямо на форму.

- <img width="672" height="382" alt="_______" src="https://github.com/user-attachments/assets/c9c451eb-e7f5-4750-a7ea-65a3f525097f" />

- На localhost:
перезапустил контейнеры со сбросом базы. Перешел по ссылке «Добавить запчасть». Ввел новое имя детали, но артикул указал старый — BC-1234 (он уже есть у тормозной колодки). Нажал кнопку сохранения. Сервер не упал. Страница перезагрузилась, и рядом с полем артикула высветился красный текст: «Запчасть с таким артикулом уже существует». Ввел уникальный артикул — деталь успешно сохранилась.

- 






# 10. Тема задачи:  Пул соединений и Maximum Pool Size
- Описание задачи: В строку подключения в docker-compose.yml добавь Maximum Pool Size=5;Minimum Pool Size=1. Напиши endpoint /Parts/PoolTest, который одновременно (Task.WhenAll) открывает 20 соединений и выполняет SELECT 1. Замерь общее время и посмотри в логах Postgres (docker compose logs db), сколько реальных соединений было установлено — должно быть не больше 5. Объясни в комментарии, почему using вокруг NpgsqlConnection не закрывает физическое соединение, а возвращает его в пул.











