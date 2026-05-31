# DbContext выполнил два SQL-запроса, но вернул один объект. Как это работает, и почему AsNoTracking меняет поведение?
- мы делаем запрос в базу первый раз и EF Core добавляет объект в Identity Map. Когда мы делаем запрос в базу данных во второй раз (EF Core не кэширует данные как Redis поэтому мы уходим в бд во второй раз),
- EF Core видит один и тот же primary key и не создает еще один объект, а перезаписывает значения если они изменились и возвращает уже существующий. Если мы добавляет к запросу .AsNoTracking() то EF Core не ведет Identity Map.

# N+1 проблема:
Вознкиает например во время lazy loading  (чтобы включить его надо на references and collections добавить virtual и добавить .UseLazyLoadingProxies() в OnConfiguring()). Когда мы лоудим из базы данных entities и нам надо из них получить reference или collection мы проходимя с помощью foreach по ним и каждый раз запрашиваем reference или collection. Таким образом EF Core создает N sql запросов и +1 изначальный запрос в базу данных. Identity Map  не является решением N+1 проблемы потому что в случае если каждый объект имеет уникальный primary key Identity Map не поможет. Придется загружать N+1 разю

# .Include(), .Select(), .AsSplitQuery()

IncludeКогда загружаешь связанные объекты без коллекций (один-к-одному), или коллекции маленькие и данных немного
Select projection Когда нужна часть данных — для списков, таблиц на UI, любых read-only endpoints
AsSplitQueryКогда нужны все данные включая большие коллекции, и cartesian explosion реальная проблема

# IQueryable vs IEnumerable

IQueryable — это описание запроса. SQL не выполняется пока ты не вызовешь ToList(), FirstOrDefault(), Count() и т.д. Каждый .Where(), .OrderBy(), .Select() просто добавляет к описанию.
IEnumerable — данные уже в памяти. Любой .Where() после этого — это C# цикл по объектам в памяти, не SQL.

# PostgreSQL Index Types

## Regular Index

```sql
CREATE INDEX ON "Orders" ("CustomerId");
```

Используется для ускорения поиска по одной колонке.

---

## Composite Index

```sql
CREATE INDEX ON "Orders" ("CustomerId", "Status");
```

### 1. Left Prefix Rule

Composite index `(A, B)` используется для:

- ✅ `WHERE A = ...`
- ✅ `WHERE A = ... AND B = ...`

Но не используется эффективно для:

- ❌ `WHERE B = ...`

Пример:

```sql
-- Использует индекс
SELECT * FROM "Orders"
WHERE "CustomerId" = 1;

-- Использует индекс
SELECT * FROM "Orders"
WHERE "CustomerId" = 1
  AND "Status" = 'Pending';

-- Обычно НЕ использует индекс
SELECT * FROM "Orders"
WHERE "Status" = 'Pending';
```

---

### 2. Порядок колонок важен

Сначала ставят колонку с более уникальными значениями (higher selectivity).

Пример:

```sql
(CustomerId, Status)
```

Почему:

- `CustomerId` → много уникальных значений
- `Status` → всего несколько значений

---

### 3. Не создавать индекс на каждую колонку

Каждый индекс:

- занимает место на диске
- замедляет: INSERT/ UPDATE/DELETE потому что PostgreSQL должен обновлять все индексы после изменения данных.

---

## Partial Index

```sql
CREATE INDEX ON "Orders" ("CustomerId")
WHERE "Status" = 'Pending';
```

Индекс создаётся только для части строк.

Полезно, если запросы часто работают с небольшим подмножеством данных.

Пример:

```sql
SELECT *
FROM "Orders"
WHERE "CustomerId" = 1
  AND "Status" = 'Pending';
```

Преимущества:

- меньше размер индекса
- быстрее обновляется
- лучше производительность для специфичных запросов

---

# ACID

## A — Atomicity

Транзакция — это всё или ничего.

Пример: перевод денег. Списание и зачисление — одна транзакция. Если зачисление не удалось — откатывается всё.

## C — Consistency

База всегда в валидном состоянии. Транзакция не может нарушить бизнес-правила и constraints.

Пример: баланс не может стать отрицательным.

## I — Isolation

Транзакции не видят промежуточные состояния друг друга. Параллельные транзакции выполняются так, будто они последовательны.

Пример: ты читаешь остаток на складе внутри транзакции — видишь 100. Потом другая транзакция меняет остаток на 40 и коммитит. Ты снова читаешь внутри той же транзакции — что увидишь зависит от уровня изоляции.

## D — Durability

Закоммиченные данные не теряются даже после краша сервера. PostgreSQL решает это через WAL.

---

# WAL (Write-Ahead Log)

Без Durability сценарий мог бы быть такой:

1. Транзакция делает COMMIT  
2. База отвечает "успех"  
3. Данные в RAM, запись на диск ещё не завершена  
4. Сервер падает  
5. После перезапуска данных нет  

## Как PostgreSQL решает это

PostgreSQL использует WAL:

1. Сначала записывает операцию в лог на диск  
2. Только потом отвечает "commit successful"  

Если сервер падает:

- читает WAL  
- восстанавливает данные  
- возвращает базу в консистентное состояние


# Pessimistic Concruccency
Когда нужно lock какую-то операцию
await using var transaction = await db.Database.BeginTransactionAsync(IsolationLevel.ReadCommitted);

var stocks = await db.Stocks.FromSqlRaw(@"SELECT * FROM ""Stocks"" WHERE ""ProductName"" = {0} ON UPDATE", productName).FirstOrDefaultAsync();
...

await db.SaveChangesAsync();

await transaction.CommitAsync();

## Unit тесты — ключевые моменты

- Всегда async Task, никогда void
- Структура AAA: Arrange → Act → Assert
- Moq: Setup настраивает поведение, Verify проверяет что метод был вызван
- throw без аргумента сохраняет stack trace, throw ex — теряет
- Setup всегда в Arrange, до Act
