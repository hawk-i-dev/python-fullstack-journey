Day 25 is React basics. Today you rebuild the expense tracker using React components and state. No backend connection yet. That comes next.

React’s docs recommend Vite for building a React app from scratch, and Create React App is deprecated for new apps, so use Vite. Sources: [React app setup](https://react.dev/learn/build-a-react-app-from-scratch), [Vite guide](https://vite.dev/guide/).

**Day 25 Goal**
You should understand:

- React component
- JSX
- `useState`
- Controlled form inputs
- Rendering lists with `.map()`
- Handling submit and click events
- Updating arrays in state

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
npm create vite@latest day-25-react-basics -- --template react
cd day-25-react-basics
npm install
code .
```

Run:

```powershell
npm run dev
```

Open the URL shown, usually:

```text
http://localhost:5173
```

Replace `src/App.jsx`:

```jsx
import { useState } from "react";
import "./App.css";

function App() {
  const [expenses, setExpenses] = useState([]);
  const [form, setForm] = useState({
    title: "",
    amount: "",
    category: "",
  });
  const [message, setMessage] = useState("");

  const total = expenses.reduce((sum, expense) => sum + expense.amount, 0);

  function handleChange(event) {
    const { name, value } = event.target;

    setForm({
      ...form,
      [name]: value,
    });
  }

  function handleSubmit(event) {
    event.preventDefault();

    const newExpense = {
      id: Date.now(),
      title: form.title.trim(),
      amount: Number(form.amount),
      category: form.category.trim(),
    };

    if (!newExpense.title || newExpense.amount <= 0 || !newExpense.category) {
      setMessage("Enter valid expense details.");
      return;
    }

    setExpenses([...expenses, newExpense]);
    setForm({ title: "", amount: "", category: "" });
    setMessage("");
  }

  function deleteExpense(expenseId) {
    const updatedExpenses = expenses.filter((expense) => expense.id !== expenseId);
    setExpenses(updatedExpenses);
  }

  return (
    <main className="container">
      <h1>React Expense Tracker</h1>

      <form onSubmit={handleSubmit}>
        <input
          name="title"
          value={form.title}
          onChange={handleChange}
          placeholder="Title"
        />

        <input
          name="amount"
          value={form.amount}
          onChange={handleChange}
          type="number"
          placeholder="Amount"
        />

        <input
          name="category"
          value={form.category}
          onChange={handleChange}
          placeholder="Category"
        />

        <button type="submit">Add Expense</button>
      </form>

      {message && <p className="error">{message}</p>}

      <h2>Total: ₹{total}</h2>

      <ul>
        {expenses.map((expense) => (
          <li key={expense.id}>
            <span>
              {expense.title} - ₹{expense.amount} - {expense.category}
            </span>

            <button onClick={() => deleteExpense(expense.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </main>
  );
}

export default App;
```

Replace `src/App.css`:

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

li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding: 12px;
  border-bottom: 1px solid #ddd;
}

.error {
  color: #dc2626;
}
```

Test:

- Add `Tea`, `20`, `Food`
- Add `Bus`, `35`, `Travel`
- Confirm total becomes `55`
- Delete one expense
- Try empty title
- Try amount `0`

**Key Concept**
This is React state:

```jsx
const [expenses, setExpenses] = useState([]);
```

Never update state directly like this:

```jsx
expenses.push(newExpense);
```

Use this instead:

```jsx
setExpenses([...expenses, newExpense]);
```

React re-renders the page only when state is updated correctly.

**Challenge**
Add a category filter input and show only matching expenses.

Commit:

```powershell
git add .
git commit -m "Day 25 learn React basics with state"
```

Day 25 is complete when you can explain why React uses state instead of manually changing HTML with `document.createElement()`.
