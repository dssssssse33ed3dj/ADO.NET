# Домашняя работа от 23.10


10 задач по ADO.NET для проекта «Автозапчасти»
1. Тема задачи: Поиск запчастей по названию и артикулу
- Описание задачи: Добавь метод SearchParts(string query) в DbHelper. Он должен возвращать список запчастей, у которых name или article содержат подстроку query (регистронезависимо). Используй параметризованный ILIKE с @q, значение - параметра — $"%{query}%". Если query пустой — вернуть все запчасти. В контроллере сделай action Search, во view — строку поиска над таблицей.
- Что было сделано мною:
открыл папку проекта autoparts-docker в VS Code. В Data/DbHelper.cs добавил метод SearchParts(query). Для защиты от SQL-инъекций применил параметризацию через @q. Поиск сделан с помощью оператора ILIKE для игнорирования регистра символов, а знаки % (подстановка $"%{query}%") нужны для поиска подстроки в любом месте полей name и article. Если входящая строка пустая, метод возвращает полный список запчастей.

- <img width="489" height="571" alt="1" src="https://github.com/user-attachments/assets/0543bcf4-7854-4a44-803f-d1774c70b406" />



2. Тема задачи: Удаление запчасти с проверкой результата
- Описание задачи: Добавь DeletePart(int id), который выполняет DELETE FROM parts WHERE id = @id и возвращает true, только если ExecuteNonQuery() вернул больше нуля. В контроллере — POST-action с антифорджери, во view — кнопка «Удалить» с confirm(). Если запись не найдена — показать сообщение через TempData.


3. Тема задачи: Статистика по складу одним запросом
- Описание задачи: Добавь GetStats(), возвращающий кортеж (int totalParts, decimal totalValue, decimal avgPrice). SQL: SELECT COUNT(*), COALESCE(SUM(price*stock),0), COALESCE(AVG(price),0) FROM parts. Читай результат через NpgsqlDataReader (не ExecuteScalar — колонок три). Выведи три карточки на отдельной странице /Parts/Stats.


4. Тема задачи: Оформление заказа в транзакции
- Описание задачи: Создай таблицу orders (id SERIAL, part_id INT, quantity INT, total NUMERIC, created_at TIMESTAMP). Метод PlaceOrder(int partId, int quantity) должен в одной транзакции: прочитать price и stock из parts, проверить что stock >= quantity, вставить запись в orders, выполнить UPDATE parts SET stock = stock - @qty WHERE id = @id, закоммитить. При нехватке товара — Rollback и false. Все NpgsqlCommand внутри транзакции обязаны получать объект tx.


5. Тема задачи: Массовое поступление товара с откатом при ошибке
- Описание задачи: Метод Restock(List<(int PartId, int AddQty)> items) в одной транзакции для каждого элемента выполняет UPDATE parts SET stock = stock + @qty WHERE id = @id. Если хотя бы один PartId не найден (ExecuteNonQuery() == 0) — откатить всю транзакцию целиком. Во view — форма с несколькими строками «id + количество».


6. Тема задачи: «Сырое» чтение через NpgsqlDataReader
- Описание задачи: Метод GetRawParts() выполняет SELECT * FROM parts и вручную раскладывает каждую строку в Dictionary<string, object?>, используя reader.FieldCount, reader.GetName(i), reader.GetValue(i) и reader.IsDBNull(i). Модель Part не использовать. Во view вывести таблицу, заголовки — из ключей словаря. Цель — прочувствовать reader без обёрток.


7. Тема задачи: DataTable через NpgsqlDataAdapter
- Описание задачи: Метод GetPartsDataTable() заполняет DataTable через NpgsqlDataAdapter и adapter.Fill(dt). Контроллер передаёт DataTable в view, view печатает строки через foreach (DataRow row in Model.Rows). Дополнительно сделай GetPartsDataSet() с двумя таблицами (parts и orders) в одном DataSet — чтобы увидеть, зачем DataSet вообще нужен.


8. Тема задачи: Асинхронная версия GetAllParts
- Описание задачи: Перепиши GetAllParts в GetAllPartsAsync(), используя OpenAsync, ExecuteReaderAsync, ReadAsync, await using. Контроллер Index должен стать async Task<IActionResult> и вызывать await _db.GetAllPartsAsync(). Убедись, что страница работает. Никаких .Result и .Wait() — только await.



9. Тема задачи: Обработка уникального ограничения (SqlState)
- Описание задачи: Добавь в Initialize() ограничение ALTER TABLE parts ADD CONSTRAINT uq_parts_article UNIQUE (article). Метод TryAddPart(Part part) возвращает (bool Ok, string? Error): пытается вставить запись, а при PostgresException с SqlState == "23505" возвращает (false, "Запчасть с таким артикулом уже существует"). В контроллере при ошибке — вернуть форму с ModelState.AddModelError. Проверь: добавь дубль артикула, убедись что сайт показывает понятную ошибку, а не 500.



10. Тема задачи:  Пул соединений и Maximum Pool Size
- Описание задачи: В строку подключения в docker-compose.yml добавь Maximum Pool Size=5;Minimum Pool Size=1. Напиши endpoint /Parts/PoolTest, который одновременно (Task.WhenAll) открывает 20 соединений и выполняет SELECT 1. Замерь общее время и посмотри в логах Postgres (docker compose logs db), сколько реальных соединений было установлено — должно быть не больше 5. Объясни в комментарии, почему using вокруг NpgsqlConnection не закрывает физическое соединение, а возвращает его в пул.



