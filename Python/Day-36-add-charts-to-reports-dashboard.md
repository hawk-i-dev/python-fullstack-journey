Regular `Day 36`: charts in the reports dashboard.

Today you turn Day 35 reports into visual charts using Recharts. Official Recharts install is `npm install recharts`, and responsive charts use `ResponsiveContainer`. Sources: [Recharts install](https://recharts.github.io/en-US/guide/), [ResponsiveContainer](https://recharts.github.io/en-US/api/ResponsiveContainer/).

**Day 36 Goal**

- Add bar chart by category
- Add pie chart by category
- Keep reports protected with JWT
- Use backend report APIs from Day 35
- Make reports easier to understand visually

Start from Day 35:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-35-react-reports-dashboard .\day-36-react-report-charts
cd day-36-react-report-charts
npm install recharts
code .
```

Run backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Replace `src/pages/ReportsPage.jsx`:

```jsx
import { useEffect, useState } from "react";
import {
  Bar,
  BarChart,
  CartesianGrid,
  Cell,
  Legend,
  Pie,
  PieChart,
  ResponsiveContainer,
  Tooltip,
  XAxis,
  YAxis,
} from "recharts";
import { getCategoryReport, getExpenseSummary } from "../api/expenses";

const COLORS = ["#2563eb", "#16a34a", "#dc2626", "#9333ea", "#f59e0b"];

function ReportsPage() {
  const [summary, setSummary] = useState({ total: 0, count: 0 });
  const [categoryReport, setCategoryReport] = useState([]);
  const [message, setMessage] = useState("");

  useEffect(() => {
    async function loadReports() {
      try {
        setSummary(await getExpenseSummary());
        setCategoryReport(await getCategoryReport());
      } catch (error) {
        setMessage(error.message);
      }
    }

    loadReports();
  }, []);

  return (
    <>
      <h1>Reports</h1>

      {message && <p className="error">{message}</p>}

      <section className="summary-grid">
        <div className="summary-card">
          <h2>₹{summary.total}</h2>
          <p>Total expense</p>
        </div>

        <div className="summary-card">
          <h2>{summary.count}</h2>
          <p>Total records</p>
        </div>
      </section>

      <section className="charts-grid">
        <div className="chart-card">
          <h2>Category Bar Chart</h2>

          <ResponsiveContainer width="100%" height={300}>
            <BarChart data={categoryReport}>
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis dataKey="category" />
              <YAxis />
              <Tooltip />
              <Bar dataKey="total" fill="#2563eb" />
            </BarChart>
          </ResponsiveContainer>
        </div>

        <div className="chart-card">
          <h2>Category Pie Chart</h2>

          <ResponsiveContainer width="100%" height={300}>
            <PieChart>
              <Pie
                data={categoryReport}
                dataKey="total"
                nameKey="category"
                outerRadius={100}
                label
              >
                {categoryReport.map((item, index) => (
                  <Cell key={item.category} fill={COLORS[index % COLORS.length]} />
                ))}
              </Pie>

              <Tooltip />
              <Legend />
            </PieChart>
          </ResponsiveContainer>
        </div>
      </section>

      <h2>By Category</h2>

      <ul>
        {categoryReport.map((item) => (
          <li key={item.category}>
            <span>{item.category}</span>
            <strong>₹{item.total}</strong>
          </li>
        ))}
      </ul>
    </>
  );
}

export default ReportsPage;
```

Add CSS to `src/App.css`:

```css
.charts-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 24px;
}

.chart-card {
  background: #f4f6f8;
  padding: 16px;
  border-radius: 8px;
}

.chart-card h2 {
  margin-top: 0;
}

@media (max-width: 700px) {
  .charts-grid,
  .summary-grid {
    grid-template-columns: 1fr;
  }
}
```

Run React:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-36-react-report-charts"
npm install
npm run dev
```

Test:

- Login
- Add expenses in multiple categories
- Open `/reports`
- Bar chart should show category totals
- Pie chart should show category share
- Login as another user and confirm charts show only that user’s data

Commit:

```powershell
git add .
git commit -m "Day 36 add charts to reports dashboard"
```

Day 36 is complete when you can explain: the backend calculates report data, and React only visualizes that data using charts.
