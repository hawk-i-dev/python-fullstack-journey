Day 11 is FastAPI basics. Today you move from command-line programs to backend APIs.

**Day 11 Goal**
You should understand:

- What an API is
- What `GET`, `POST`, `PUT`, `DELETE` mean
- What a route/endpoint is
- What request body and response body are
- How FastAPI auto-generates API docs
- How backend code receives JSON and returns JSON

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-11-fastapi-basics
cd day-11-fastapi-basics
code .
```

Install FastAPI:

```powershell
python -m pip install fastapi uvicorn
```

Create `main.py`:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI(title="Expense API")

expenses = []
next_expense_id = 1


class ExpenseCreate(BaseModel):
    title: str = Field(min_length=1)
    amount: float = Field(gt=0)
    category: str = Field(min_length=1)


class ExpenseUpdate(BaseModel):
    title: str = Field(min_length=1)
    amount: float = Field(gt=0)
    category: str = Field(min_length=1)


@app.get("/")
def home():
    return {"message": "Expense API is running"}


@app.get("/expenses")
def list_expenses():
    return expenses


@app.get("/expenses/{expense_id}")
def get_expense(expense_id: int):
    for expense in expenses:
        if expense["id"] == expense_id:
            return expense

    raise HTTPException(status_code=404, detail="Expense not found")


@app.post("/expenses", status_code=201)
def add_expense(expense_data: ExpenseCreate):
    global next_expense_id

    expense = {
        "id": next_expense_id,
        "title": expense_data.title,
        "amount": expense_data.amount,
        "category": expense_data.category,
    }

    expenses.append(expense)
    next_expense_id += 1

    return expense


@app.put("/expenses/{expense_id}")
def update_expense(expense_id: int, expense_data: ExpenseUpdate):
    for expense in expenses:
        if expense["id"] == expense_id:
            expense["title"] = expense_data.title
            expense["amount"] = expense_data.amount
            expense["category"] = expense_data.category
            return expense

    raise HTTPException(status_code=404, detail="Expense not found")


@app.delete("/expenses/{expense_id}")
def delete_expense(expense_id: int):
    for expense in expenses:
        if expense["id"] == expense_id:
            expenses.remove(expense)
            return {"message": "Expense deleted"}

    raise HTTPException(status_code=404, detail="Expense not found")
```

Run the server:

```powershell
uvicorn main:app --reload
```

Open this in browser:

```text
http://127.0.0.1:8000/docs
```

That `/docs` page is Swagger UI. It lets you test your API without building frontend yet.

**Test These**
Use `/docs` and test in this order:

1. `GET /`
2. `GET /expenses`
3. `POST /expenses`

Use this JSON:

```json
{
  "title": "Tea",
  "amount": 20,
  "category": "Food"
}
```

4. `GET /expenses`
5. `GET /expenses/1`
6. `PUT /expenses/1`
7. `DELETE /expenses/1`
8. `GET /expenses/1`

After delete, `GET /expenses/1` should return `404`.

**Important Concepts**
`@app.get("/expenses")` means this function runs when the browser/client sends a `GET` request to `/expenses`.

`ExpenseCreate` defines the JSON body expected by the API.

`Field(gt=0)` means amount must be greater than zero.

`HTTPException(status_code=404)` means the requested record was not found.

Current limitation: data is stored in a Python list, so it disappears when the server restarts. That is expected. Day 12 will connect this API to PostgreSQL.

**Challenge**
Add this endpoint:

```python
@app.get("/expenses/category/{category_name}")
def get_expenses_by_category(category_name: str):
    matches = []

    for expense in expenses:
        if expense["category"].lower() == category_name.lower():
            matches.append(expense)

    return matches
```

Then test:

```text
GET /expenses/category/Food
```

Commit:

```powershell
git status
git add .
git commit -m "Day 11 build first FastAPI app"
```

Day 11 is complete when you can explain the difference between a Python CLI app and a backend API.
