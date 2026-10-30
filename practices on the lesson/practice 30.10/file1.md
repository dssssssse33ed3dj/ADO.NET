# Практика по 15 билетам на паре по ADO.NET + PostgreSQL
- Суть работы: на практике перед завчетом применить ADO.NET с PostgreSQL/Docker - прогнать все ключевые темы через 4 эндпоинта.

Что конкретно нужно было:
**Общие условия для всех билетов:**
- Время: 1,5 часа
- БД: PostgreSQL (Docker)
- Проект: ASP.NET Core Minimal API (.NET 8), пакеты `Npgsql` и `Dapper`
- В **каждом** билете студент должен реализовать **все темы**: соединение, команды, параметры, транзакции, DataSet, DataAdapter, DataTable, асинхронность, пулы, Dapper
- Каркасы содержат только сигнатуры эндпоинтов и пустые тела — без подсказок и решений

---

## Общие файлы (для всех билетов)

**`MyApp.csproj`**
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <AssemblyName>MyApp</AssemblyName>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Npgsql" Version="8.0.5" />
    <PackageReference Include="Dapper" Version="2.1.35" />
  </ItemGroup>
</Project>
```

**`Dockerfile`**
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
ENV ASPNETCORE_URLS=http://+:80
EXPOSE 80
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

**`docker-compose.yml`**
```yaml
version: '3.8'
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: shopdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports: ["5432:5432"]
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
  app:
    build: .
    environment:
      CONN_STRING: "Host=db;Port=5432;Database=shopdb;Username=postgres;Password=postgres;Pooling=true;Minimum Pool Size=1;Maximum Pool Size=20"
    ports: ["8080:80"]
    depends_on: [db]
```

---


## Билет 5 — «HR: сотрудники и отделы»

**Задание:** Эндпоинты:
- `GET /parallel` — параллельно `Task.WhenAll` получить count(сотрудники) и count(отделы) с замером времени (соединение + команда + async + пул);
- `POST /raise` — повышение зарплаты отдела в транзакции с параметрами;
- `GET /employees` — через Dapper;
- `GET /employees/table` — отчёт через `DataSet`/`DataAdapter`/`DataTable`.

**`init.sql`**
```sql
CREATE TABLE departments (id SERIAL PRIMARY KEY, name VARCHAR(100) NOT NULL);
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    dep_id INT REFERENCES departments(id),
    name VARCHAR(100) NOT NULL,
    salary NUMERIC(10,2) NOT NULL
);
INSERT INTO departments (name) VALUES ('IT'), ('HR');
INSERT INTO employees (dep_id, name, salary) VALUES
    (1, 'Иван', 80000),
    (2, 'Мария', 70000);
```

**`Program.cs` (каркас)**
```csharp
using System.Data;
using System.Diagnostics;
using Dapper;
using Npgsql;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
string cs = Environment.GetEnvironmentVariable("CONN_STRING")!;

app.MapGet("/parallel", async () =>
{
    return Results.Ok();
});

app.MapPost("/raise", async (Raise r) =>
{
    return Results.Ok();
});

app.MapGet("/employees", async () =>
{
    return Results.Ok();
});

app.MapGet("/employees/table", () =>
{
    return Results.Ok();
});

app.Run();
record Raise(int DepId, decimal Delta);
```

---

# Что было сделано мною: 
- Поднял PostgreSQL и app через docker-compose (сервисы `db` + `app`)
- Схему залил через `init.sql` в `docker-entrypoint-initdb.d`
- Строку подключения передал через `CONN_STRING` (Pooling=true)
- Реализовал все 4 эндпоинта (код в `Program.cs`)
- Проверил curl'ом: все отвечают 200 OK с ожидаемыми данными


<img width="1012" height="1372" alt="1" src="https://github.com/user-attachments/assets/d621b873-651c-4b0e-a1c5-cb24fa9944a5" />
<img width="973" height="1353" alt="2" src="https://github.com/user-attachments/assets/4da9eff3-cafb-4256-ae49-8e206cb5d2d8" />


### Покрытие тем
| Тема | Где |
|---|---|
| Соединение | все 4 |
| Команды | /parallel, /raise |
| Параметры | /raise |
| Транзакция | /raise |
| Async | /parallel, /raise, /employees |
| Пул | /parallel |
| Dapper | /employees |
| DataSet/DataAdapter/DataTable | /employees/table |

### Как запустить
```
docker compose down -v
docker compose up --build
curl.exe http://localhost:8080/parallel

```

### Что в итоге вывелось:

Запросы выполнились и параллельности есть на скриншоте, но elapsedMs: 0 - потому что на 2 строках в таблицах COUNT выполняется меньше миллисекунды, и Stopwatch возвращает 0 (он округляет до целых). Параллельность видна архитектурно: два соединения, две команды и Task.WhenAll и если бы делал await t1; await t2; - было бы то же самое по времени, но это уже последовательно, а не параллельно


<img width="477" height="157" alt="3" src="https://github.com/user-attachments/assets/8260ccbc-add3-4f2d-b698-c5c124f4bf04" />



## Что осталось / вопросы
- В конспекте упоминается tx.IsCompleted — в Npgsql 8 его нет, убрал
- DataAdapter нет в конспектах — писал по классическому шаблону ADO.NET
