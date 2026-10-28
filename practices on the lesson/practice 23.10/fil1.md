
# Практическая работа от 23.10 с первой пары: Docker: Магазин автозапчастей на C# + PostgreSQL

- Цель задачи: Научиться запускать сайты в Docker на примере веб-приложения на C# (ASP.NET Core) из данного репозитория и базы данных PostgreSQL, чтобы они работали вместе в изолированных контейнерах и автоматически связывались друг с другом. А также повторить структуру файлов из данного репозитория - https://github.com/yaroslavrychikin-crypto/ado.net/blob/main/practica1.md и переписать код и методы для файла DBHelper.cs
  
- Описание работы / что сделал: Сначала через консоль создал и почистил от мусора папки проекта, после чего дописал код в пустой шаблон DbHelper.cs. Затем  написал Dockerfile и настроил docker-compose.yml. В итоге сайт и база данных теперь запускаются одной командой, а проверка healthcheck следит, чтобы сайт не включался раньше, чем полностью загрузится база.

---


- Как и в прошлом дз на сегодня, сперва я в cmd создал при помощи команд на скриншоте проект и папку:
- <img width="1093" height="547" alt="231" src="https://github.com/user-attachments/assets/bbe78c79-0509-4296-9694-7482745bb467" />

- После чего я при помощи команд в консоли Visual Studio Code ниже создал и почистил от мусора струкутру папок:
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


### Структура проекта и назначение файлов

#### 1. Окружение и управление Docker

| Файл / Папка | Роль и назначение для задания |
| :--- | :--- |
| `docker-compose.yml` | Центр управления. Компилируеет два контейнера (сайт на C# и БД PostgreSQL) |
| `Dockerfile` | Инструкция к сборке приложения. Пошагово объясняет Docker, как скомпилировать код C# и упаковать его в контейнер |
| `appsettings.json` | Блокнот с настройками. Хранит строку подключения к PostgreSQL с данными |
| `AutoPartsApp.csproj` | Паспорт проекта. Здесь прописана библиотека Npgsql для работы с PostgreSQL. |
| `Program.cs` | Точка старта. Первым запускает код приложения, включает веб-сервер и сразу вызывает DbHelper.Initialize(). |

#### Компоненты приложения 

| Компонент или Файл | Тип и для чего нужен |
| :--- | :--- |
| `Models/Part.cs` | Model (Модель). Описывает запчасть в виде объекта ID, имМЕНИ и т.д. |
| `Views/Parts/` <br> ├ `Index.cshtml` <br> ├ `Details.cshtml` <br> └ `Create.cshtml` | View (Интерфейс). HTML-страницы сайта с кодом C#:<br>• Index - витрина товаров.<br>• Details - карточка одной запчасти.<br>• Create - форма добавления товара. |
| `Controllers/PartsController.cs` | Controller. Принимает клики пользователя, вызывает методы из DbHelper и отдает результат в Views |
| `Data/DbHelper.cs` | Data Layer (База). Главный файл задания. Пишет чистый SQL через Npgsql чтобы читать, добавлять и обновлять товары |

---

## Data/DbHelper.cs — **пустой каркас**

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


- Мой код:
<img width="788" height="1281" alt="908888888888888888888888888888888888" src="https://github.com/user-attachments/assets/d2a338e5-ba5e-4753-80ca-4ed705b68ff7" />
<img width="655" height="1240" alt="11111вааааааа" src="https://github.com/user-attachments/assets/5a10b6da-8da3-4550-bdab-790559124864" />

# Что было изменено по сравнению с тем шаблоном в этом коде файла DbHelper.cs: 

Я взял шаблон и написал логику работы с базой данных. Чтобы программа сама закрывала соединения, я все подключения обернул в блоки using (без этого соединения будут открыты и забивать память).
В первом методе Initialize я сначала добавил создание таблицы parts, если её ещё нет в базе и после я поставил проверку: если в таблице пусто, то мы загружаем в неё начальные  данне, а если данные есть, мы ничего не трогаем, чтобы они не дублировались при каждом перезапуске. В методах вывода GetAllParts и GetPartById я сделал чтение строк через NpgsqlDataReader библиотеку, при этом отдельно проверил поле описания - если оно в базе пустое, программа не упадет с ошибкой, а просто запишет в C#-модельку null. Для добавления деталей и покупки я не стал вставлять переменные в текст SQL-запроса, а безопасные параметры через AddWithValue (защищает сайт от хакерских SQL-инъекций) А в методе покупки DecreaseStock я написал запрос UPDATE с условием stock >= @qty - БД сама проверяет это условие в момент клика, поэтому у нас не получится продать больше деталей, чем есть на складе, даже если покупку нажмут несколько человек одновременно



## Запуск

```bash
docker compose up --build
```

- У меня возникла ошибка после строки **docker compose up --build**. Я прогнал ее через нейронку и она написала мне так - **Эта ошибка повторяется потому, что у серверов Debian время спешит ровно на 1 секунду относительно системных часов твоего Docker-контейнера. Очистка кэша здесь не поможет, так как Linux упорно считает файл обновлений «еще не созданным».**.
- И я взял и исправил строчку в Dockerfile **RUN apt-get update && apt-get install -y libgssapi-krb5-2** на --> **RUN apt-get -o Acquire::Check-Valid-Until=false update && apt-get install -y libgssapi-krb5-2**
 <img width="1182" height="505" alt="прддддд" src="https://github.com/user-attachments/assets/f090ae47-b203-4af7-8c2a-54b9332d5252" />

- В итог дело было в 1 секунде серверов, далее я исправил строчку и сервер запустился локально
- <img width="2558" height="1438" alt="___________________" src="https://github.com/user-attachments/assets/ca206828-4596-41e2-a555-038005cece22" />


Сайт: **http://localhost:8080** 



---
