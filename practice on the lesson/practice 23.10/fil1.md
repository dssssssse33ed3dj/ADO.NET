
# Практическая работа от 23.10 с первой пары: Docker: Магазин автозапчастей на C# + PostgreSQL

- Цель задачи: Научиться запускать сайты в Docker на примере веб-приложения на C# (ASP.NET Core) и базы данных PostgreSQL, чтобы они работали вместе в изолированных контейнерах и автоматически связывались друг с другом.
- Описание работы / что сделал: Сначала через консоль создал и почистил от мусора папки проекта, после чего дописал код в пустой шаблон DbHelper.cs. Затем  написал Dockerfile и настроил docker-compose.yml. В итоге сайт и база данных теперь запускаются одной командой, а проверка healthcheck следит, чтобы сайт не включался раньше, чем полностью загрузится база.

---

## Структура проекта

```
autoparts-docker/
├── docker-compose.yml
├── Dockerfile
├── AutoPartsApp.csproj
├── Program.cs
├── appsettings.json
├── Controllers/
│   └── PartsController.cs
├── Models/
│   └── Part.cs
├── Data/
│   └── DbHelper.cs          ← пустой, пишешь сам
└── Views/
    └── Parts/
        ├── Index.cshtml
        ├── Details.cshtml
        └── Create.cshtml
```

- Как и в прошлом дз на сегодня я в cmd создал при помощи команд проект и папку:
- <img width="1093" height="547" alt="231" src="https://github.com/user-attachments/assets/bbe78c79-0509-4296-9694-7482745bb467" />

- После чего я при помощи команд ниже создал и почистил от мусора струкутру папок:
```
:: 1. Создаем новые папки
mkdir Data
mkdir Views\Parts

:: 2. Создаем пустые файлы конфигурации и кода
echo. > docker-compose.yml
echo. > Dockerfile
echo. > Controllers\PartsController.cs
echo. > Models\Part.cs
echo. > Data\DbHelper.cs
echo. > Views\Parts\Index.cshtml
echo. > Views\Parts\Details.cshtml
echo. > Views\Parts\Create.cshtml

:: 3. Удаляем старый шаблонный мусор от прошлого проекта (Home), чтобы не мешался
del /q Controllers\HomeController.cs
del /q Models\ErrorViewModel.cs
del /q Views\Shared\Error.cshtml
del /q Views\_ViewImports.cshtml
del /q Views\_ViewStart.cshtml
rmdir /s /q Views\Home

```

- Вышло вот так:
- <img width="1341" height="1327" alt="9088888888888" src="https://github.com/user-attachments/assets/a19e636c-78d1-4d47-9118-110a10680e47" />


---

## 1. docker-compose.yml

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: autopartsdb
      POSTGRES_USER: partsuser
      POSTGRES_PASSWORD: partspass
    ports:
      - "5433:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U partsuser -d autopartsdb"]
      interval: 5s
      timeout: 5s
      retries: 10
      start_period: 10s

  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ConnectionStrings__Default=Host=db;Port=5432;Database=autopartsdb;Username=partsuser;Password=partspass;SslMode=Disable
    depends_on:
      db:
        condition: service_healthy

volumes:
  pgdata:
```

---

## 2. Dockerfile

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish
RUN apt-get update && apt-get install -y libgssapi-krb5-2
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENV ASPNETCORE_HTTP_PORTS=8080
ENTRYPOINT ["dotnet", "AutoPartsApp.dll"]
```

---

## 3. AutoPartsApp.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Npgsql" Version="8.0.5" />
  </ItemGroup>
</Project>
```

---

## 4. Models/Part.cs

```csharp
namespace AutoPartsApp.Models;

public class Part
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;        // Название запчасти
    public string Article { get; set; } = string.Empty;     // Артикул
    public string Manufacturer { get; set; } = string.Empty; // Производитель
    public decimal Price { get; set; }                      // Цена
    public int Stock { get; set; }                          // Остаток на складе
    public string? Description { get; set; }                // Описание
}
```

---

## 5. Data/DbHelper.cs — **пустой каркас**

Оставлены только сигнатуры, поля и `TODO`. Тебе нужно самому реализовать каждый метод через `NpgsqlConnection` / `NpgsqlCommand` / `NpgsqlDataReader`.

```csharp
using AutoPartsApp.Models;

namespace AutoPartsApp.Data;

public class DbHelper
{
    private readonly string _connectionString;

    public DbHelper(string connectionString)
    {
        _connectionString = connectionString;
    }

    // TODO: создать таблицу parts, если её нет, и заполнить тестовыми данными,
    // если таблица пуста. Вызывается один раз при старте приложения.
    // Подсказка: CREATE TABLE IF NOT EXISTS, SERIAL PRIMARY KEY, затем SELECT COUNT(*)
    public void Initialize()
    {
        throw new NotImplementedException();
    }

    // TODO: вернуть все запчасти (SELECT ... ORDER BY id)
    public List<Part> GetAllParts()
    {
        throw new NotImplementedException();
    }

    // TODO: вернуть одну запчасть по id или null, если не найдена
    public Part? GetPartById(int id)
    {
        throw new NotImplementedException();
    }

    // TODO: добавить запчасть (INSERT), параметры через AddWithValue,
    // description может быть null → DBNull.Value
    public void AddPart(Part part)
    {
        throw new NotImplementedException();
    }

    // TODO: атомарно уменьшить остаток при покупке:
    // UPDATE parts SET stock = stock - @qty WHERE id = @id AND stock >= @qty
    // Вернуть true, если строка обновилась (ExecuteNonQuery > 0), иначе false
    public bool DecreaseStock(int partId, int quantity)
    {
        throw new NotImplementedException();
    }
}
```

Что нужно написать — и в каком порядке:

1. `Initialize()` — DDL + сидинг.
2. `GetAllParts()` — `SELECT` + `NpgsqlDataReader`, цикл `while (reader.Read())`.
3. `GetPartById(int id)` — `SELECT ... WHERE id = @id`, обработка `IsDBNull(..)` для `description`.
4. `AddPart(Part part)` — `INSERT` с параметрами.
5. `DecreaseStock(...)` — `UPDATE ... WHERE stock >= @qty`.

Каркас SQL-схемы, которую нужно создать в `Initialize()` (можешь брать как ориентир):

```sql
CREATE TABLE IF NOT EXISTS parts (
    id           SERIAL PRIMARY KEY,
    name         VARCHAR(200) NOT NULL,
    article      VARCHAR(50)  NOT NULL,
    manufacturer VARCHAR(100) NOT NULL,
    price        NUMERIC(18,2) NOT NULL,
    stock        INTEGER NOT NULL DEFAULT 0,
    description  VARCHAR(500)
);
```

Тестовые данные для сидинга (пример):

```sql
INSERT INTO parts (name, article, manufacturer, price, stock, description) VALUES
('Тормозная колодка', 'BC-1234', 'Bosch', 2500.00, 40, 'Передняя, для легковых авто'),
('Масляный фильтр', 'OF-5678', 'Mann', 850.00, 120, 'Универсальный'),
('Свеча зажигания', 'SP-9012', 'NGK', 450.00, 200, 'Иридиевая');
```

- Мой код:
<img width="788" height="1281" alt="908888888888888888888888888888888888" src="https://github.com/user-attachments/assets/d2a338e5-ba5e-4753-80ca-4ed705b68ff7" />
<img width="655" height="1240" alt="11111вааааааа" src="https://github.com/user-attachments/assets/5a10b6da-8da3-4550-bdab-790559124864" />

# Что было изменено по сравнению с тем шаблоном в этом коде файла DbHelper.cs: 

Я взял шаблон и написал логику работы с базой данных. Чтобы программа сама закрывала соединения, я все подключения обернул в блоки using (без этого соединения будут открыты и забивать память).
В первом методе Initialize я сначала добавил создание таблицы parts, если её ещё нет в базе и после я поставил проверку: если в таблице пусто, то мы загружаем в неё начальные  данне, а если данные есть, мы ничего не трогаем, чтобы они не дублировались при каждом перезапуске. В методах вывода GetAllParts и GetPartById я сделал чтение строк через NpgsqlDataReader библиотеку, при этом отдельно проверил поле описания - если оно в базе пустое, программа не упадет с ошибкой, а просто запишет в C#-модельку null. Для добавления деталей и покупки я не стал вставлять переменные в текст SQL-запроса, а безопасные параметры через AddWithValue (защищает сайт от хакерских SQL-инъекций) А в методе покупки DecreaseStock я написал запрос UPDATE с условием stock >= @qty - БД сама проверяет это условие в момент клика, поэтому у нас не получится продать больше деталей, чем есть на складе, даже если покупку нажмут несколько человек одновременно


---

## 6. Controllers/PartsController.cs

Контроллер **рабочий**, чтобы после реализации `DbHelper` всё сразу заработало.

```csharp
using Microsoft.AspNetCore.Mvc;
using AutoPartsApp.Data;
using AutoPartsApp.Models;

namespace AutoPartsApp.Controllers;

public class PartsController : Controller
{
    private readonly DbHelper _db;

    public PartsController(DbHelper db)
    {
        _db = db;
    }

    // GET /Parts
    public IActionResult Index()
    {
        var parts = _db.GetAllParts();
        return View(parts);
    }

    // GET /Parts/Details/5
    public IActionResult Details(int id)
    {
        var part = _db.GetPartById(id);
        if (part == null) return NotFound();
        return View(part);
    }

    // GET /Parts/Create
    public IActionResult Create() => View();

    // POST /Parts/Create
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(Part part)
    {
        if (ModelState.IsValid)
        {
            _db.AddPart(part);
            return RedirectToAction(nameof(Index));
        }
        return View(part);
    }

    // POST /Parts/Buy/5
    [HttpPost]
    public IActionResult Buy(int id, int quantity = 1)
    {
        var success = _db.DecreaseStock(id, quantity);
        TempData["Message"] = success
            ? $"Запчасть куплена! Остаток уменьшен на {quantity}."
            : "Недостаточно запчастей на складе.";
        return RedirectToAction(nameof(Index));
    }
}
```

---

## 7. Program.cs

```csharp
using AutoPartsApp.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();

var connectionString = builder.Configuration.GetConnectionString("Default")
    ?? throw new InvalidOperationException("Connection string not found");

builder.Services.AddSingleton(new DbHelper(connectionString));

var app = builder.Build();

// Инициализация БД с ретраями — как только ты реализуешь DbHelper.Initialize(), это заработает
using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<DbHelper>();
    var retries = 10;
    while (retries > 0)
    {
        try
        {
            db.Initialize();
            Console.WriteLine("База данных инициализирована.");
            break;
        }
        catch (NotImplementedException)
        {
            Console.WriteLine("DbHelper.Initialize() ещё не реализован — пропускаю инициализацию БД.");
            break; // чтобы приложение не падало, пока ты пишешь код
        }
        catch (Exception ex)
        {
            retries--;
            Console.WriteLine($"Ожидание PostgreSQL... Осталось попыток: {retries}. {ex.Message}");
            Thread.Sleep(3000);
        }
    }
}

app.UseStaticFiles();
app.UseRouting();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Parts}/{action=Index}/{id?}");

app.Run();
```

> Пока `DbHelper` пустой, сайт **запустится** (спасибо отдельному `catch (NotImplementedException)`), но при заходе на `/Parts` будет `500`, потому что `GetAllParts()` кинет `NotImplementedException`. Это нормально — по мере реализации методы будут оживать.

---

## 8. appsettings.json

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Port=5432;Database=autopartsdb;Username=partsuser;Password=partspass;SslMode=Disable;GSS Encryption Mode=Disable"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

---

## 9. Views/Parts/Index.cshtml

```cshtml
@model List<AutoPartsApp.Models.Part>
@{
    ViewData["Title"] = "Каталог автозапчастей";
    var message = TempData["Message"] as string;
}

<h1>Каталог автозапчастей</h1>

@if (!string.IsNullOrEmpty(message))
{
    <div style="padding:10px; background:#e7f5e7; border:1px solid #3c3; margin:10px 0;">
        @message
    </div>
}

<p><a asp-action="Create">+ Добавить запчасть</a></p>

<table border="1" cellpadding="8" cellspacing="0">
    <tr>
        <th>Id</th><th>Название</th><th>Артикул</th><th>Производитель</th>
        <th>Цена</th><th>Остаток</th><th></th>
    </tr>
    @foreach (var p in Model)
    {
        <tr>
            <td>@p.Id</td>
            <td><a asp-action="Details" asp-route-id="@p.Id">@p.Name</a></td>
            <td>@p.Article</td>
            <td>@p.Manufacturer</td>
            <td>@p.Price.ToString("C")</td>
            <td>@p.Stock</td>
            <td>
                <form asp-action="Buy" method="post" style="display:inline;">
                    <input type="hidden" name="id" value="@p.Id" />
                    <input type="number" name="quantity" value="1" min="1" max="@p.Stock" style="width:50px;" />
                    <button type="submit" @(p.Stock == 0 ? "disabled" : "")>Купить</button>
                </form>
            </td>
        </tr>
    }
</table>
```

---

## 10. Views/Parts/Details.cshtml

```cshtml
@model AutoPartsApp.Models.Part
@{
    ViewData["Title"] = Model.Name;
}

<h1>@Model.Name</h1>
<p><b>Артикул:</b> @Model.Article</p>
<p><b>Производитель:</b> @Model.Manufacturer</p>
<p><b>Цена:</b> @Model.Price.ToString("C")</p>
<p><b>На складе:</b> @Model.Stock</p>
<p><b>Описание:</b> @Model.Description</p>

<form asp-action="Buy" method="post">
    <input type="hidden" name="id" value="@Model.Id" />
    <input type="number" name="quantity" value="1" min="1" max="@Model.Stock" />
    <button type="submit" @(Model.Stock == 0 ? "disabled" : "")>Купить</button>
</form>

<p><a asp-action="Index">← Назад к каталогу</a></p>
```

---

## 11. Views/Parts/Create.cshtml

```cshtml
@model AutoPartsApp.Models.Part
@{
    ViewData["Title"] = "Добавить запчасть";
}

<h1>Добавить запчасть</h1>

<form asp-action="Create" method="post">
    @Html.AntiForgeryToken()
    <p><label>Название</label><br /><input asp-for="Name" required maxlength="200" /></p>
    <p><label>Артикул</label><br /><input asp-for="Article" required maxlength="50" /></p>
    <p><label>Производитель</label><br /><input asp-for="Manufacturer" required maxlength="100" /></p>
    <p><label>Цена</label><br /><input asp-for="Price" type="number" step="0.01" min="0" required /></p>
    <p><label>Остаток</label><br /><input asp-for="Stock" type="number" min="0" required /></p>
    <p><label>Описание</label><br /><textarea asp-for="Description" maxlength="500"></textarea></p>
    <button type="submit">Сохранить</button>
    <a asp-action="Index">Отмена</a>
</form>
```

---

## 12. Быстрое создание структуры

### Linux / macOS

```bash
mkdir autoparts-docker && cd autoparts-docker
mkdir -p Controllers Models Data Views/Parts
touch docker-compose.yml Dockerfile AutoPartsApp.csproj Program.cs appsettings.json \
      Controllers/PartsController.cs Models/Part.cs Data/DbHelper.cs \
      Views/Parts/Index.cshtml Views/Parts/Details.cshtml Views/Parts/Create.cshtml
```

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Path autoparts-docker | Out-Null
Set-Location autoparts-docker
New-Item -ItemType Directory -Force -Path Controllers, Models, Data, Views\Parts | Out-Null
$files = @("docker-compose.yml","Dockerfile","AutoPartsApp.csproj","Program.cs","appsettings.json",
           "Controllers\PartsController.cs","Models\Part.cs","Data\DbHelper.cs",
           "Views\Parts\Index.cshtml","Views\Parts\Details.cshtml","Views\Parts\Create.cshtml")
foreach ($f in $files) { New-Item -ItemType File -Force -Path $f | Out-Null }
```

---

## 13. Запуск

```bash
docker compose up --build
```

- У меня возникла опять ошибка ниже. Я прогнал ее через нейронку и она написала мне так - **Эта ошибка повторяется потому, что у серверов Debian время спешит ровно на 1 секунду относительно системных часов твоего Docker-контейнера. Очистка кэша здесь не поможет, так как Linux упорно считает файл обновлений «еще не созданным».**.
- И я взял и исправил строчку **RUN apt-get update && apt-get install -y libgssapi-krb5-2** на --> **RUN apt-get -o Acquire::Check-Valid-Until=false update && apt-get install -y libgssapi-krb5-2**
 <img width="1182" height="505" alt="прддддд" src="https://github.com/user-attachments/assets/f090ae47-b203-4af7-8c2a-54b9332d5252" />

- В итог дело реально было в этой спешке в 1 секунду серверов, я исправил строчку и сервер запустился локально
- <img width="2558" height="1438" alt="___________________" src="https://github.com/user-attachments/assets/ca206828-4596-41e2-a555-038005cece22" />





Сайт: **http://localhost:8080** → редирект на `/Parts`.

Пока `DbHelper` пустой — увидишь `NotImplementedException` при заходе в каталог. Как только реализуешь методы — всё заработает без пересборки `docker-compose.yml` (только пересборка образа `web`):

```bash
docker compose up -d --build web
```

---

## Что делать дальше (порядок реализации `DbHelper`)

1. **`Initialize()`** — начни с `NpgsqlConnection`, `Open()`, `NpgsqlCommand` с `CREATE TABLE IF NOT EXISTS parts (...)`, `ExecuteNonQuery()`. Затем `SELECT COUNT(*)` → `ExecuteScalar()` → если `0`, выполни `INSERT`-сидинг.
2. **`GetAllParts()`** — `SELECT id, name, article, manufacturer, price, stock, description FROM parts ORDER BY id`, читай через `NpgsqlDataReader` в цикле, собирай `List<Part>`.
3. **`GetPartById()`** — тот же `SELECT` с `WHERE id = @id`, `Parameters.AddWithValue("@id", id)`, верни первый результат или `null`.
4. **`AddPart()`** — `INSERT INTO parts (...) VALUES (@name, @article, @manufacturer, @price, @stock, @description)`, `description`: `(object?)part.Description ?? DBNull.Value`.
5. **`DecreaseStock()`** — `UPDATE parts SET stock = stock - @qty WHERE id = @id AND stock >= @qty`, `return cmd.ExecuteNonQuery() > 0;`.

