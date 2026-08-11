Day 24 is connecting frontend to backend using JavaScript `fetch()`. Yesterday the UI stored expenses only in browser memory. Today the UI will call your FastAPI API.

**Day 24 Goal**
You should understand:

- What `fetch()` does
- `async` / `await`
- Calling `GET`, `POST`, `DELETE` APIs from JavaScript
- CORS
- Frontend/backend separation
- Browser UI talking to FastAPI

Use your Day 14 API first because it has no login. Authentication frontend will come later.

Start backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-14-expense-api-review"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Keep that terminal running.

Open a second PowerShell:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
mkdir day-24-frontend-api-fetch
cd day-24-frontend-api-fetch
code .
```

Create:

```text
day-24-frontend-api-fetch/
├── index.html
├── styles.css
└── script.js
```

Before the browser can call FastAPI, enable CORS in your Day 14 `app/main.py`:

```python
from fastapi.middleware.cors import CORSMiddleware
```

Add this after `app = FastAPI(...)`:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Now create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Expense API Frontend</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <main class="container">
        <h1>Expense Tracker</h1>

        <form id="expense-form">
            <input id="title" type="text" placeholder="Title" required>
            <input id="amount" type="number" placeholder="Amount" required>
            <input id="category" type="text" placeholder="Category" required>
            <button type="submit">Add Expense</button>
        </form>

        <p id="message"></p>

        <h2>Total: ₹<span id="total">0</span></h2>

        <ul id="expense-list"></ul>
    </main>

    <script src="script.js"></script>
</body>
</html>
```

Create `styles.css`:

```css
body {
    font-family: Arial, sans-serif;
    background: #f4f6f8;
    margin: 0;
}

.container {
    max-width: 720px;
    margin: 40px auto;
    background: white;
    padding: 24px;
    border-radius: 8px;
}

form {
    display: grid;
    gap: 12px;
    margin-bottom: 16px;
}

input,
button {
    padding: 12px;
    font-size: 16px;
}

button {
    background: #2563eb;
    color: white;
    border: none;
    cursor: pointer;
}

.delete-button {
    background: #dc2626;
}

li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 16px;
    padding: 12px;
    border-bottom: 1px solid #ddd;
}

#message {
    color: #dc2626;
}
```

Create `script.js`:

```javascript
const API_URL = "http://127.0.0.1:8000";

const form = document.getElementById("expense-form");
const titleInput = document.getElementById("title");
const amountInput = document.getElementById("amount");
const categoryInput = document.getElementById("category");
const expenseList = document.getElementById("expense-list");
const totalElement = document.getElementById("total");
const messageElement = document.getElementById("message");

async function loadExpenses() {
    const response = await fetch(`${API_URL}/expenses`);
    const expenses = await response.json();

    renderExpenses(expenses);
}

function renderExpenses(expenses) {
    expenseList.innerHTML = "";

    let total = 0;

    for (const expense of expenses) {
        total += expense.amount;

        const item = document.createElement("li");

        item.innerHTML = `
            <span>${expense.title} - ₹${expense.amount} - ${expense.category}</span>
        `;

        const deleteButton = document.createElement("button");
        deleteButton.textContent = "Delete";
        deleteButton.className = "delete-button";

        deleteButton.addEventListener("click", function () {
            deleteExpense(expense.id);
        });

        item.appendChild(deleteButton);
        expenseList.appendChild(item);
    }

    totalElement.textContent = total;
}

async function addExpense(expense) {
    const response = await fetch(`${API_URL}/expenses`, {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify(expense),
    });

    if (!response.ok) {
        messageElement.textContent = "Failed to add expense.";
        return;
    }

    messageElement.textContent = "";
    form.reset();
    loadExpenses();
}

async function deleteExpense(expenseId) {
    const response = await fetch(`${API_URL}/expenses/${expenseId}`, {
        method: "DELETE",
    });

    if (!response.ok) {
        messageElement.textContent = "Failed to delete expense.";
        return;
    }

    loadExpenses();
}

form.addEventListener("submit", function (event) {
    event.preventDefault();

    const expense = {
        title: titleInput.value.trim(),
        amount: Number(amountInput.value),
        category: categoryInput.value.trim(),
    };

    if (!expense.title || expense.amount <= 0 || !expense.category) {
        messageElement.textContent = "Enter valid expense details.";
        return;
    }

    addExpense(expense);
});

loadExpenses();
```

Open `index.html` in browser.

Test:

- Add Tea, 20, Food
- Add Bus, 35, Travel
- Refresh browser
- Data should still appear because it comes from PostgreSQL
- Delete one expense
- Check Swagger `/docs` and confirm it is deleted there too

Commit:

```powershell
git add .
git commit -m "Day 24 connect frontend to FastAPI with fetch"
```

Day 24 is complete when you can explain this flow:

`Browser form -> JavaScript fetch -> FastAPI endpoint -> PostgreSQL -> FastAPI JSON response -> JavaScript updates HTML`.
