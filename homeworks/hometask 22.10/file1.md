### Домашняя работа от 22.10 (повторение заданий из репозитория ado net преподавателя)

**Цель работы**: Развернуть веб-приложение ASP.NET Core MVC с использованием технологии ADO.NET (Npgsql) и реляционной базы данных PostgreSQL в изолированной контейнерной среде Docker.

**Описание работы / что сделал**
* **Настроил окружение:** Установил Docker Desktop и активировал подсистему WSL для поддержки Linux-контейнеров на Windows.
* **Создал архитектуру:** Сгенерировал каркас C# проекта через .NET CLI, настроил структуру папок (MVC + Data) и удалил лишние шаблонные файлы.
* **Написал код данных:** Создал модель Product и класс DbHelper с чистыми SQL-запросами на ADO.NET для инициализации таблиц и управления товарами.
* **Сконфигурировал Docker:** Написал файлы Dockerfile (сборка приложения на .NET 10.0) и docker-compose.yml (оркестрация сайта и СУБД PostgreSQL с общим томом pgdata).
* **Успешно запустил:** Развернул мультиконтейнерное приложение командой `docker compose up -d`, проверив стабильную работу базы данных и веб-интерфейса.

---

- Сперва я скачал с офиц. сайта Docker Desktop, а затем перезагрузил пк

 <img width="1528" height="923" alt="111111111111111" src="https://github.com/user-attachments/assets/abb5889b-0237-4f04-b371-5807b1bcf003" />


- После я нажал комбинацию WIN + R и ввёл команду **wsl.exe --update**, поскольку он выдавал ошибку такую что значит что подсистема WSL не установлена (это технология, которая позволяет запускать полноценную систему Linux прямо внутри Windows бех виртуалки)
- <img width="1024" height="679" alt="image" src="https://github.com/user-attachments/assets/966da89f-c734-4480-9eb9-a5c294e34418" />

- Чтобы не бьло ошибок в Visual Studio я скачал расширения
-  <img width="410" height="1017" alt="image" src="https://github.com/user-attachments/assets/b04f1827-9caa-40c5-adc7-712c68efe3c2" />


## Создание проекта
```cmd
# Создать MVC-проект в папке shop-docker
dotnet new mvc -n ShopApp -o shop-docker — создаёт готовый шаблон сайта (MVC-проект) на C# с именем ShopApp и складывает все его файлы в новую папку shop-docker.
cd shop-docker — переносит нас (командную строку) внутрь этой созданной папки, чтобы все следующие действия и команды выполнялись уже внутри проекта.

# Добавить пакет Npgsql
dotnet add package Npgsql --version 8.0.5

```
- <img width="2543" height="1272" alt="99999999" src="https://github.com/user-attachments/assets/66df69bf-fb1f-4b2a-8c9b-1be31cccfbd7" />

```
# Удалить стандартные шаблонные файлы, которые не нужны
Views/Home Views/Shared/Error.cshtml Views/_ViewImports.cshtml Views/_ViewStart.cshtml
Controllers/HomeController.cs Models/ErrorViewModel.cs

# Создать наши папки и файлы
Data Views/Products
Data/DbHelper.cs \
      Controllers/ProductsController.cs \
      Models/Product.cs \
      Views/Products/Index.cshtml \
      Views/Products/Details.cshtml \
      Views/Products/Create.cshtml \
      docker-compose.yml \
      Dockerfile
```
## Что меняется по сравнению с SQL Server версией

| Компонент | SQL Server | PostgreSQL |
|---|---|---|
| Образ | `mssql/server` | `postgres:16-alpine` |
| Провайдер ADO.NET | `Microsoft.Data.SqlClient` | `Npgsql` |
| Connection string | `Server=db,1433;...` | `Host=db;Port=5432;...` |
| Параметры в SQL | `@Name` | `@name` (или `$1`) — Npgsql понимает `@` |
| Автоинкремент | `IDENTITY(1,1)` | `SERIAL` / `GENERATED ALWAYS AS IDENTITY` |
| Строки | `NVARCHAR` | `VARCHAR` / `TEXT` |
| Проверка готовности | `sqlcmd` | `pg_isready` |

---

## Структура проекта

```
shop-docker/
├── docker-compose.yml
├── Dockerfile
├── ShopApp.csproj
├── Program.cs
├── appsettings.json
├── Controllers/
│   └── ProductsController.cs
├── Models/
│   └── Product.cs
├── Data/
│   └── DbHelper.cs
└── Views/
    └── Products/
        ├── Index.cshtml
        ├── Details.cshtml
        └── Create.cshtml
```

- <img width="1511" height="1282" alt="343434343434343434343434" src="https://github.com/user-attachments/assets/29d25e12-28df-4985-9b8a-5f26a1c205d6" />

- Тут я создал чистый каркас веб-приложения (ASP.NET Core MVC) с помощью .NET CLI и собрал правильную архитектуру: Настроил файловую структуру строго по инструкции - создал файлы для Docker-контейнеров, контроллеров, моделей и базы данных с помощью команды в консоли Visual Studio


---



## Запуск

```bash
docker compose up --build
```

- После того, как я вставил все команды по инструкции в файлы свои и выполнил команду выше вылезла ошибка, что в файле настроек проекта указана версия .NET 10.0, а Dockerfile пытается собрать его с помощью старого образа .NET 8.0. Я исправил это одной строчкой измененной в файле Dockerfile: 
<img width="1700" height="471" alt="32222222222221пмаолттттт" src="https://github.com/user-attachments/assets/5cb99cfb-6d2b-4a28-b2ad-531e98f96ab3" />



После старта:
- Сайт: **http://localhost:8080** (редирект на `/Products`)
- Postgres: `localhost:5432`, база `shopdb`, пользователь `shopuser`, пароль `shoppass`

- Сайт успешно встал, все получилось:
  <img width="1000" height="528" alt="234ва" src="https://github.com/user-attachments/assets/69880f1d-7e43-4c33-b773-c3906db2b318" />


- И проект успешно контейнеризирован и запущен в Docker Desktop. Все сервисы приложения работают стабильно в Docker Desktop:
<img width="1615" height="942" alt="56565656565656565656" src="https://github.com/user-attachments/assets/2f2ad6e5-bdb9-4818-9ca7-7f16bfe48755" />


Проверить базу напрямую:

```bash
docker compose exec db psql -U shopuser -d shopdb -c "SELECT * FROM products;"
```

- <img width="1571" height="483" alt="23232323111______________" src="https://github.com/user-attachments/assets/0634d1f7-6b43-4b80-b24a-314dab1e83bb" />
- Я перезапустил с помощью команды **docker compose up -d** контейнеры в фоновом режиме в VS и все вывелось.

---

## Что «проработано» (сценарии)

1. **Инициализация БД** — `CREATE TABLE IF NOT EXISTS` + сидинг, только если таблица пуста (идемпотентно при перезапуске).
2. **Ожидание БД** — healthcheck + retry в `Program.cs` на случай гонки.
3. **Чтение списка** — `NpgsqlDataReader`, корректная обработка `DBNull`.
4. **Чтение одной записи** — параметризованный запрос, `NotFound()` при отсутствии.
5. **Создание** — параметризованный INSERT, `DBNull.Value` для nullable-поля.
6. **Покупка / уменьшение остатка** — атомарный `UPDATE ... WHERE stock >= @qty`: если товара не хватает, `ExecuteNonQuery` вернёт `0`, и пользователь увидит сообщение. Это защищает от «ухода в минус» при параллельных запросах.
7. **Изоляция данных** — том `pgdata`, данные переживают `docker compose down` (но не `down -v`).
8. **Параметризация везде** — SQL-инъекции невозможны.

---

