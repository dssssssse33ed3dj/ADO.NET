### Конспекты за 22.10 и 21.10

Есть два способа работы с PostgreSQL - напрямую через команды или через таблицы в памяти. Первый способ вручную мы процедуры вызываем через CALL, функции через SELECT, а одиночные значения забирают методом ExecuteScalar(). Второй способ скачивает данные в объект DataSet и позволяет работать с ними локально. Можно создать структуру таблицы (DataTable), настроить столбцы (DataColumn), ключи (PrimaryKey) и добавлять новые строки (DataRow). Плюс - NpgsqlDataAdapter сам управляет подключением, а с NpgsqlCommandBuilder позволяет одной командой adapter.Update() автоматически отправить все локальные изменения обратно в БД

Ключевой код к конспекту:

```
// 1.Параметры, создание и вызов процедур/функций
Command.Parameters.AddWithValue("Name", "Ret");
Command.Parameters.Add("FD", NpgsqlDbType.Integer).Value = 0;

using (var v = new NpgsqlCommand(createProc, con)) { con.Open(); v.ExecuteNonQuery(); con.Close(); }
using (var cmd = new NpgsqlCommand("CALL get_fa(@p_n)", con)) { cmd.Parameters.AddWithValue("p_n", "dd"); con.Open(); cmd.ExecuteNonQuery(); con.Close(); }
// int i = Convert.ToInt32(cmd.ExecuteScalar());
var ca = new NpgsqlCommand("SELECT fgrty()", con); ca.CommandType = System.Data.CommandType.StoredProcedure;

// 2.Автоматическая работа с данными и синхронизация через адаптер
NpgsqlDataAdapter adapter = new NpgsqlDataAdapter(sql, connection);
DataSet ds = new DataSet();
adapter.Fill(ds);

DataTable dt = ds.Tables[0];
DataRow newRow = dt.NewRow();
newRow["Name"] = "Rick"; newRow["Age"] = 24; dt.Rows.Add(newRow);
dt.Rows[0]["Age"] = 17;

NpgsqlCommandBuilder commandBuilder = new NpgsqlCommandBuilder(adapter);
adapter.Update(ds);
ds.Clear(); adapter.Fill(ds);

// 3.Конструирование структуры таблицы и колонок в памяти
DataSet usersSet = new DataSet("UsersSet");
DataTable users = new DataTable("Users");
usersSet.Tables.Add(users);

DataColumn idColumn = new DataColumn("Id", Type.GetType("System.Int32"));
idColumn.Unique = true; idColumn.AllowDBNull = false; idColumn.AutoIncrement = true; idColumn.AutoIncrementSeed = 1; idColumn.AutoIncrementStep = 1;
users.Columns.Add(idColumn);
users.PrimaryKey = new DataColumn[] { users.Columns["Id"] };

DataRow row = users.NewRow(); row.ItemArray = new object[] { null, "Tom", 36 }; users.Rows.Add(row);
```

---
