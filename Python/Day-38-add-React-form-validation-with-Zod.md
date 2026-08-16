`Day 38`: React form validation with React Hook Form + Zod.

Today you replace manual form checks with professional frontend validation. React Hook Form handles form state/errors, Zod defines validation rules, and `@hookform/resolvers` connects them. Sources: [React Hook Form](https://www.react-hook-form.com/), [@hookform/resolvers Zod docs](https://www.npmjs.com/package/%40hookform/resolvers).

**Day 38 Goal**

- Add field-level validation errors
- Validate register/login/category/expense forms
- Disable submit while saving
- Reduce manual `if (!title...)` checks
- Keep backend validation too

Start from Day 37:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-37-react-custom-hooks .\day-38-react-form-validation
cd day-38-react-form-validation
npm install react-hook-form zod @hookform/resolvers
code .
```

Create `src/validation/schemas.js`:

```javascript
import { z } from "zod";

export const registerSchema = z.object({
  username: z.string().min(3, "Username must be at least 3 characters."),
  email: z.string().email("Enter a valid email."),
  password: z.string().min(6, "Password must be at least 6 characters."),
});

export const loginSchema = z.object({
  username: z.string().min(1, "Username is required."),
  password: z.string().min(1, "Password is required."),
});

export const categorySchema = z.object({
  name: z.string().min(1, "Category name is required."),
});

export const expenseSchema = z.object({
  title: z.string().min(1, "Title is required."),
  amount: z.number().positive("Amount must be greater than 0."),
  category_id: z.number().positive("Select a category."),
});
```

Example: update `src/pages/LoginPage.jsx`:

```jsx
import { zodResolver } from "@hookform/resolvers/zod";
import { useForm } from "react-hook-form";
import { useNavigate } from "react-router";
import { loginUser, saveToken } from "../api/auth";
import { loginSchema } from "../validation/schemas";

function LoginPage() {
  const navigate = useNavigate();
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({ resolver: zodResolver(loginSchema) });

  async function onSubmit(data) {
    const result = await loginUser(data.username, data.password);
    saveToken(result.access_token);
    navigate("/profile");
  }

  return (
    <>
      <h1>Login</h1>

      <form onSubmit={handleSubmit(onSubmit)}>
        <input {...register("username")} placeholder="Username" />
        {errors.username && <p className="error">{errors.username.message}</p>}

        <input {...register("password")} type="password" placeholder="Password" />
        {errors.password && <p className="error">{errors.password.message}</p>}

        <button disabled={isSubmitting} type="submit">
          {isSubmitting ? "Logging in..." : "Login"}
        </button>
      </form>
    </>
  );
}

export default LoginPage;
```

For expense amount and category fields, use `valueAsNumber`:

```jsx
<input
  type="number"
  {...register("amount", { valueAsNumber: true })}
  placeholder="Amount"
/>

<select {...register("category_id", { valueAsNumber: true })}>
  <option value="">Select category</option>
  {categories.map((category) => (
    <option key={category.id} value={category.id}>
      {category.name}
    </option>
  ))}
</select>
```

Run backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Run React:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-38-react-form-validation"
npm run dev
```

Test:

- Empty login form shows field errors
- Invalid email fails on register
- Expense amount `0` fails
- Expense without category fails
- Valid forms still call backend correctly

Commit:

```powershell
git add .
git commit -m "Day 38 add React form validation with Zod"
```

Day 38 is complete when you can explain: Zod defines the rules, React Hook Form manages form state, and the resolver connects them. Backend validation is still required because frontend validation can be bypassed.
