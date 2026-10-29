# Конспект за 29.10

Структура

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
