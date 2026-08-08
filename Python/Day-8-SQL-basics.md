Day 8 is SQL basics with PostgreSQL. This is the start of backend/database work.

**Day 8 Goal**
You should understand:

- What a database is
- What a table is
- Rows and columns
- Primary key
- `CREATE`, `INSERT`, `SELECT`, `UPDATE`, `DELETE`
- Why backend apps use databases instead of JSON files

Open PostgreSQL.

Best option on Windows:

```text
Start Menu > SQL Shell (psql)
```

Use these values:

```text
Server: localhost
Database: postgres
Port: 5432
Username: postgres
Password: your PostgreSQL password
```

Create your Day 8 database:

```sql
CREATE DATABASE python_fullstack_day8;
```

Connect to it:

```sql
\c python_fullstack_day8
```

Create an `expenses` table:

```sql
CREATE TABLE expenses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    amount NUMERIC(10, 2) NOT NULL,
    category VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Insert data:

```sql
INSERT INTO expenses (title, amount, category)
VALUES ('Tea', 20.00, 'Food');

INSERT INTO expenses (title, amount, category)
VALUES ('Bus ticket', 35.00, 'Travel');

INSERT INTO expenses (title, amount, category)
VALUES ('Notebook', 60.00, 'Study');
```

Read data:

```sql
SELECT * FROM expenses;
```

Filter data:

```sql
SELECT * FROM expenses
WHERE category = 'Food';
```

Sort data:

```sql
SELECT * FROM expenses
ORDER BY amount DESC;
```

Calculate total:

```sql
SELECT SUM(amount) AS total_expense
FROM expenses;
```

Group by category:

```sql
SELECT category, SUM(amount) AS total
FROM expenses
GROUP BY category;
```

Update data:

```sql
UPDATE expenses
SET amount = 25.00
WHERE id = 1;
```

Delete data:

```sql
DELETE FROM expenses
WHERE id = 2;
```

Check final result:

```sql
SELECT * FROM expenses;
```

**Important Concepts**

`id SERIAL PRIMARY KEY` means every row gets a unique number automatically.

`NOT NULL` means that column cannot be empty.

`VARCHAR(100)` means text with max 100 characters.

`NUMERIC(10, 2)` is good for money values like `2500.75`.

`created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP` automatically saves when the row was created.

**Your Challenge**
Create another table named `contacts`:

```sql
CREATE TABLE contacts (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    phone VARCHAR(20) NOT NULL,
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Then do:

- Insert 3 contacts
- Select all contacts
- Search one contact by name
- Update one phone number
- Delete one contact
- Select all contacts again

Commit note for today:

```powershell
git status
git add .
git commit -m "Day 8 learn PostgreSQL basics"
```

Day 8 is complete when you can explain this clearly: JSON file stores app data in a file, but PostgreSQL stores structured data in tables and lets us search, filter, update, and connect data efficiently.
