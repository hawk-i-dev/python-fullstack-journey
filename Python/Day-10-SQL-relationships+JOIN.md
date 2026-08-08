Day 10 is PostgreSQL relationships + `JOIN`. This is a core backend skill. Real apps do not keep everything in one table.

**Day 10 Goal**
You should understand:

- Foreign keys
- One-to-many relationship
- `JOIN`
- `GROUP BY`
- Why database design matters
- Reading joined data from Python

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-10-sql-relationships
cd day-10-sql-relationships
code .
```

Open `psql` and create database:

```sql
CREATE DATABASE python_fullstack_day10;
\c python_fullstack_day10
```

Create two related tables:

```sql
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE expenses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    amount NUMERIC(10, 2) NOT NULL,
    category_id INTEGER NOT NULL REFERENCES categories(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Insert sample data:

```sql
INSERT INTO categories (name)
VALUES ('Food'), ('Travel'), ('Study');

INSERT INTO expenses (title, amount, category_id)
VALUES
('Tea', 20.00, 1),
('Bus ticket', 35.00, 2),
('Python book', 500.00, 3),
('Lunch', 120.00, 1);
```

Now run your first `JOIN`:

```sql
SELECT expenses.id, expenses.title, expenses.amount, categories.name AS category
FROM expenses
JOIN categories ON expenses.category_id = categories.id
ORDER BY expenses.id;
```

Total by category:

```sql
SELECT categories.name, SUM(expenses.amount) AS total
FROM expenses
JOIN categories ON expenses.category_id = categories.id
GROUP BY categories.name
ORDER BY total DESC;
```

**Important Concept**
`expenses.category_id` stores the category number.

`categories.id` is the real category.

That means this:

```text
expenses.category_id = categories.id
```

connects both tables.

Now create `db.py`:

```python
import psycopg

DB_NAME = "python_fullstack_day10"
DB_USER = "postgres"
DB_PASSWORD = "your_password_here"
DB_HOST = "localhost"
DB_PORT = 5432


def get_connection():
    return psycopg.connect(
        dbname=DB_NAME,
        user=DB_USER,
        password=DB_PASSWORD,
        host=DB_HOST,
        port=DB_PORT,
    )
```

Create `reports.py`:

```python
from db import get_connection


def list_expenses_with_categories():
    sql = """
        SELECT expenses.id, expenses.title, expenses.amount, categories.name
        FROM expenses
        JOIN categories ON expenses.category_id = categories.id
        ORDER BY expenses.id
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql)
            return cursor.fetchall()


def get_total_by_category():
    sql = """
        SELECT categories.name, SUM(expenses.amount)
        FROM expenses
        JOIN categories ON expenses.category_id = categories.id
        GROUP BY categories.name
        ORDER BY SUM(expenses.amount) DESC
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql)
            return cursor.fetchall()
```

Create `main.py`:

```python
from reports import get_total_by_category, list_expenses_with_categories


def show_expenses():
    expenses = list_expenses_with_categories()

    for expense in expenses:
        print(f"{expense[0]}. {expense[1]} - {expense[2]} - {expense[3]}")


def show_totals():
    totals = get_total_by_category()

    for total in totals:
        print(f"{total[0]}: {total[1]}")


def main():
    print("\nExpenses With Categories")
    print("------------------------")
    show_expenses()

    print("\nTotal By Category")
    print("-----------------")
    show_totals()


if __name__ == "__main__":
    main()
```

Run:

```powershell
python main.py
```

**Your Challenge**
Add a function named `add_expense(title, amount, category_name)`.

Rules:

- If category exists, use its `id`
- If category does not exist, create it first
- Then insert the expense using `category_id`

This is how real backend systems work: user enters `"Food"`, but database stores the relationship using `category_id`.

Commit:

```powershell
git status
git add .
git commit -m "Day 10 learn SQL relationships and joins"
```

Day 10 is complete when you can explain why `category_id` is better than storing category text repeatedly in every expense row.
