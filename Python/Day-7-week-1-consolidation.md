Day 7 is Week 1 consolidation. Today you build one cleaner Python mini-project using everything so far: functions, lists, dictionaries, files, JSON, OOP, validation, and tests.

**Day 7 Goal**
You should be able to build a small real-world CLI app without blindly copying code.

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-07-expense-tracker
cd day-07-expense-tracker
code .
```

Create this structure:

```text
day-07-expense-tracker/
├── expense.py
├── expense_tracker.py
├── main.py
└── test_expense_tracker.py
```

Create `expense.py`:

```python
class Expense:
    def __init__(self, title, amount, category):
        self.title = title
        self.amount = amount
        self.category = category

    def to_dict(self):
        return {
            "title": self.title,
            "amount": self.amount,
            "category": self.category,
        }
```

Create `expense_tracker.py`:

```python
import json

from expense import Expense

FILE_NAME = "expenses.json"


class ExpenseTracker:
    def __init__(self):
        self.expenses = []

    def add_expense(self, title, amount, category):
        if not title.strip():
            return False

        if amount <= 0:
            return False

        if not category.strip():
            return False

        expense = Expense(title.strip(), amount, category.strip())
        self.expenses.append(expense)
        return True

    def get_total_expense(self):
        total = 0

        for expense in self.expenses:
            total += expense.amount

        return total

    def get_expenses_by_category(self, category):
        matches = []

        for expense in self.expenses:
            if expense.category.lower() == category.lower():
                matches.append(expense)

        return matches

    def save_expenses(self):
        data = []

        for expense in self.expenses:
            data.append(expense.to_dict())

        with open(FILE_NAME, "w") as file:
            json.dump(data, file, indent=4)

    def load_expenses(self):
        try:
            with open(FILE_NAME, "r") as file:
                data = json.load(file)

            for item in data:
                expense = Expense(item["title"], item["amount"], item["category"])
                self.expenses.append(expense)

        except FileNotFoundError:
            self.expenses = []
        except json.JSONDecodeError:
            self.expenses = []
```

Create `main.py`:

```python
from expense_tracker import ExpenseTracker


def show_menu():
    print("\nExpense Tracker")
    print("1. Add expense")
    print("2. List expenses")
    print("3. Show total")
    print("4. Search by category")
    print("5. Exit")


def list_expenses(expenses):
    if not expenses:
        print("No expenses found.")
        return

    for index, expense in enumerate(expenses, start=1):
        print(f"{index}. {expense.title} - {expense.amount} - {expense.category}")


def main():
    tracker = ExpenseTracker()
    tracker.load_expenses()

    while True:
        show_menu()
        choice = input("Choose an option: ").strip()

        if choice == "1":
            title = input("Title: ")
            category = input("Category: ")

            try:
                amount = float(input("Amount: "))
            except ValueError:
                print("Amount must be a number.")
                continue

            if tracker.add_expense(title, amount, category):
                tracker.save_expenses()
                print("Expense added.")
            else:
                print("Invalid expense details.")

        elif choice == "2":
            list_expenses(tracker.expenses)

        elif choice == "3":
            print(f"Total expense: {tracker.get_total_expense()}")

        elif choice == "4":
            category = input("Category: ")
            matches = tracker.get_expenses_by_category(category)
            list_expenses(matches)

        elif choice == "5":
            print("Goodbye.")
            break

        else:
            print("Invalid choice.")


if __name__ == "__main__":
    main()
```

Create `test_expense_tracker.py`:

```python
from expense_tracker import ExpenseTracker


def test_add_valid_expense():
    tracker = ExpenseTracker()

    result = tracker.add_expense("Tea", 20, "Food")

    assert result is True
    assert len(tracker.expenses) == 1


def test_reject_empty_title():
    tracker = ExpenseTracker()

    result = tracker.add_expense("", 20, "Food")

    assert result is False
    assert len(tracker.expenses) == 0


def test_reject_zero_amount():
    tracker = ExpenseTracker()

    result = tracker.add_expense("Tea", 0, "Food")

    assert result is False


def test_get_total_expense():
    tracker = ExpenseTracker()
    tracker.add_expense("Tea", 20, "Food")
    tracker.add_expense("Bus", 30, "Travel")

    assert tracker.get_total_expense() == 50


def test_get_expenses_by_category():
    tracker = ExpenseTracker()
    tracker.add_expense("Tea", 20, "Food")
    tracker.add_expense("Bus", 30, "Travel")

    matches = tracker.get_expenses_by_category("food")

    assert len(matches) == 1
    assert matches[0].title == "Tea"
```

Run the app:

```powershell
python main.py
```

Run tests:

```powershell
pytest
```

Format and check:

```powershell
black .
ruff check .
```

**Challenge**
Add option `6. Delete expense`.

Rules:

- Show all expenses with numbers
- Ask which expense number to delete
- Validate the number
- Delete it
- Save updated expenses

Commit:

```powershell
git status
git add .
git commit -m "Day 7 build tested expense tracker"
```

Day 7 is complete when you can explain why `ExpenseTracker` contains the business logic and `main.py` only handles user input/output. Tomorrow we move toward backend foundations: SQL.
