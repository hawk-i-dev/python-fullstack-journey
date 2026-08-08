Day 9 is Python + PostgreSQL connection. Today your Python code stops using JSON and starts talking to a real database.

**Day 9 Goal**
You should understand:

- Connecting Python to PostgreSQL
- Installing a database driver
- Running SQL from Python
- Passing values safely with parameters
- Committing database changes
- Reading rows from database results

Start in PowerShell:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-09-python-postgres
cd day-09-python-postgres
code .
```

Install PostgreSQL driver:

```powershell
python -m pip install psycopg[binary]
```

Create database in `psql` if you did not already:

```sql
CREATE DATABASE python_fullstack_day9;
```

Connect:

```sql
\c python_fullstack_day9
```

Create table:

```sql
CREATE TABLE expenses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    amount NUMERIC(10, 2) NOT NULL,
    category VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Create `db.py`:

```python
import psycopg

DB_NAME = "python_fullstack_day9"
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

Replace `your_password_here` with your PostgreSQL password.

Create `expense_repository.py`:

```python
from db import get_connection


def add_expense(title, amount, category):
    sql = """
        INSERT INTO expenses (title, amount, category)
        VALUES (%s, %s, %s)
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (title, amount, category))


def list_expenses():
    sql = """
        SELECT id, title, amount, category, created_at
        FROM expenses
        ORDER BY id
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql)
            return cursor.fetchall()


def delete_expense(expense_id):
    sql = """
        DELETE FROM expenses
        WHERE id = %s
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (expense_id,))
            return cursor.rowcount
```

Create `main.py`:

```python
from expense_repository import add_expense, delete_expense, list_expenses


def show_menu():
    print("\nExpense Database App")
    print("1. Add expense")
    print("2. List expenses")
    print("3. Delete expense")
    print("4. Exit")


def show_expenses():
    expenses = list_expenses()

    if not expenses:
        print("No expenses found.")
        return

    for expense in expenses:
        print(
            f"{expense[0]}. {expense[1]} - {expense[2]} - "
            f"{expense[3]} - {expense[4]}"
        )


def main():
    while True:
        show_menu()
        choice = input("Choose an option: ").strip()

        if choice == "1":
            title = input("Title: ").strip()
            category = input("Category: ").strip()

            try:
                amount = float(input("Amount: "))
            except ValueError:
                print("Amount must be a number.")
                continue

            if not title or amount <= 0 or not category:
                print("Invalid expense details.")
                continue

            add_expense(title, amount, category)
            print("Expense added.")

        elif choice == "2":
            show_expenses()

        elif choice == "3":
            try:
                expense_id = int(input("Expense ID to delete: "))
            except ValueError:
                print("Expense ID must be a number.")
                continue

            deleted_count = delete_expense(expense_id)

            if deleted_count == 0:
                print("No expense found with that ID.")
            else:
                print("Expense deleted.")

        elif choice == "4":
            print("Goodbye.")
            break

        else:
            print("Invalid choice.")


if __name__ == "__main__":
    main()
```

Run:

```powershell
python main.py
```

**Critical Detail**
This is safe:

```python
cursor.execute(sql, (title, amount, category))
```

This is unsafe:

```python
cursor.execute(f"INSERT INTO expenses VALUES ('{title}', {amount}, '{category}')")
```

The safe version protects against SQL injection and handles quotes correctly.

**Your Challenge**
Add option `5. Search by category`.

Repository function idea:

```python
def find_expenses_by_category(category):
    sql = """
        SELECT id, title, amount, category, created_at
        FROM expenses
        WHERE LOWER(category) = LOWER(%s)
        ORDER BY id
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (category,))
            return cursor.fetchall()
```

Commit:

```powershell
git status
git add .
git commit -m "Day 9 connect Python to PostgreSQL"
```

Day 9 is complete when you can explain:

- What `psycopg` does
- Why database credentials are in `db.py`
- Why `%s` placeholders are safer than f-strings in SQL
- Why `connection` and `cursor` are needed
