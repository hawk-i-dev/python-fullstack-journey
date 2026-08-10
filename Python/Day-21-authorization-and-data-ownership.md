Day 21 is authorization and data ownership. Authentication answers “who are you?” Authorization answers “what data are you allowed to access?”

**Day 21 Goal**
You should understand:

- Public routes vs protected routes
- Current logged-in user
- `user_id` / ownership
- Preventing one user from seeing another user’s expenses
- Filtering database queries by current user

Start by copying Day 20:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
Copy-Item -Recurse .\day-20-auth-jwt .\day-21-user-owned-expenses
cd day-21-user-owned-expenses
code .
```

Update `.env`:

```env
DB_NAME=python_fullstack_day21
DB_USER=postgres
DB_PASSWORD=your_password_here
DB_HOST=localhost
DB_PORT=5432
JWT_SECRET_KEY=your_generated_secret_key
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Create database:

```sql
CREATE DATABASE python_fullstack_day21;
```

Update `app/models.py`.

Add ownership relationship:

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    email: Mapped[str] = mapped_column(String(100), unique=True, index=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(default=True)

    expenses: Mapped[list["Expense"]] = relationship(back_populates="owner")
```

Update `Expense`:

```python
class Expense(Base):
    __tablename__ = "expenses"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    title: Mapped[str] = mapped_column(String(100))
    amount: Mapped[float] = mapped_column(Numeric(10, 2))
    category_id: Mapped[int] = mapped_column(ForeignKey("categories.id"))
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), index=True)
    created_at: Mapped[str] = mapped_column(DateTime, server_default=func.now())

    category: Mapped[Category] = relationship(back_populates="expenses")
    owner: Mapped[User] = relationship(back_populates="expenses")
```

Run migration:

```powershell
alembic revision --autogenerate -m "add user ownership to expenses"
alembic upgrade head
```

Now update `app/routers/expenses.py`.

Import `User` and `get_current_user`:

```python
from app.models import Category, Expense, User
from app.security import get_current_user
```

Update `POST /expenses`:

```python
@router.post("", response_model=ExpenseResponse, status_code=201)
def create_expense(
    expense_data: ExpenseCreate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    category = db.query(Category).filter(Category.id == expense_data.category_id).first()

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")

    expense = Expense(
        title=expense_data.title.strip(),
        amount=expense_data.amount,
        category_id=expense_data.category_id,
        user_id=current_user.id,
    )

    db.add(expense)
    db.commit()
    db.refresh(expense)

    return expense
```

Update `GET /expenses`:

```python
@router.get("", response_model=list[ExpenseResponse])
def list_expenses(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    return (
        db.query(Expense)
        .filter(Expense.user_id == current_user.id)
        .order_by(Expense.id)
        .all()
    )
```

Update `GET /expenses/{expense_id}`:

```python
@router.get("/{expense_id}", response_model=ExpenseResponse)
def get_expense(
    expense_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    expense = (
        db.query(Expense)
        .filter(Expense.id == expense_id, Expense.user_id == current_user.id)
        .first()
    )

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    return expense
```

Update `DELETE /expenses/{expense_id}` the same way:

```python
expense = (
    db.query(Expense)
    .filter(Expense.id == expense_id, Expense.user_id == current_user.id)
    .first()
)
```

Run:

```powershell
uvicorn app.main:app --reload
```

Test in Swagger:

1. Register user `hari`
2. Login as `hari`
3. Authorize
4. Create category
5. Create expense
6. List expenses
7. Register another user `testuser`
8. Login as `testuser`
9. List expenses

`testuser` should not see `hari`’s expenses.

**Challenge**
Protect these too:

- `PUT /expenses/{expense_id}`
- `GET /categories/{category_id}/expenses`

Rule: users should only update, delete, or view expenses that belong to them.

Commit:

```powershell
git add .
git commit -m "Day 21 add user ownership and authorization"
```

Day 21 is complete when you can explain: every protected expense query must include both `expense_id` and `current_user.id`, otherwise users may access data that does not belong to them.
