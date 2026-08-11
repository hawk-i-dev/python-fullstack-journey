Day 23 starts frontend foundations. Before React, you need plain HTML, CSS, and JavaScript DOM basics. React becomes much easier after this.

**Day 23 Goal**
You should understand:

- HTML page structure
- CSS styling
- JavaScript variables and arrays
- DOM selection
- Event handling
- Rendering data on a page
- Adding data from a form

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
mkdir day-23-frontend-basics
cd day-23-frontend-basics
code .
```

Create this structure:

```text
day-23-frontend-basics/
├── index.html
├── styles.css
└── script.js
```

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Expense Tracker UI</title>
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
    max-width: 700px;
    margin: 40px auto;
    background: white;
    padding: 24px;
    border-radius: 8px;
}

form {
    display: grid;
    gap: 12px;
    margin-bottom: 24px;
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

li {
    display: flex;
    justify-content: space-between;
    padding: 12px;
    border-bottom: 1px solid #ddd;
}
```

Create `script.js`:

```javascript
const form = document.getElementById("expense-form");
const titleInput = document.getElementById("title");
const amountInput = document.getElementById("amount");
const categoryInput = document.getElementById("category");
const expenseList = document.getElementById("expense-list");
const totalElement = document.getElementById("total");

const expenses = [];

function renderExpenses() {
    expenseList.innerHTML = "";

    let total = 0;

    for (const expense of expenses) {
        total += expense.amount;

        const item = document.createElement("li");
        item.innerHTML = `
            <span>${expense.title} - ${expense.category}</span>
            <strong>₹${expense.amount}</strong>
        `;

        expenseList.appendChild(item);
    }

    totalElement.textContent = total;
}

form.addEventListener("submit", function (event) {
    event.preventDefault();

    const expense = {
        title: titleInput.value.trim(),
        amount: Number(amountInput.value),
        category: categoryInput.value.trim(),
    };

    if (!expense.title || expense.amount <= 0 || !expense.category) {
        alert("Enter valid expense details.");
        return;
    }

    expenses.push(expense);
    renderExpenses();

    form.reset();
});
```

Open `index.html` in browser.

**Your Task**

Test this:

- Add Tea, 20, Food
- Add Bus, 35, Travel
- Add Book, 500, Study
- Confirm total updates correctly
- Try empty title
- Try amount `0`

**Challenge**

Add a delete button for each expense.

Hint inside `renderExpenses()`:

```javascript
const deleteButton = document.createElement("button");
deleteButton.textContent = "Delete";

deleteButton.addEventListener("click", function () {
    expenses.splice(index, 1);
    renderExpenses();
});
```

You will need this loop style:

```javascript
expenses.forEach(function (expense, index) {
    // create list item here
});
```

Commit:

```powershell
git add .
git commit -m "Day 23 build frontend expense tracker UI"
```

Day 23 is complete when you can explain: HTML gives structure, CSS gives style, and JavaScript changes the page when the user interacts with it.
