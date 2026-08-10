Day 20 is authentication with JWT. Today you add login/register behavior to your FastAPI backend.

FastAPI’s current security guide uses `OAuth2PasswordBearer`, `PyJWT`, and `pwdlib` for password hashing, so we’ll use that stack. Sources: [FastAPI JWT security guide](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/), [FastAPI security first steps](https://fastapi.tiangolo.com/tutorial/security/first-steps/), [PyJWT docs](https://pyjwt.readthedocs.io/en/stable/).

**Day 20 Goal**
You should understand:

- User registration
- Password hashing
- Login with username/password
- JWT access tokens
- Protected API routes
- `Authorization: Bearer <token>`

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-20-auth-jwt
cd day-20-auth-jwt
code .
```

Install:

```powershell
python -m pip install fastapi uvicorn sqlalchemy psycopg[binary] pydantic-settings alembic pyjwt pwdlib[argon2] python-multipart
```

Use your Day 18/19 project structure, then add authentication files.

Add to `.env`:

```env
JWT_SECRET_KEY=generate_a_real_secret_key
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Generate a secret key:

```powershell
python -c "import secrets; print(secrets.token_hex(32))"
```

Add those fields to `app/config.py`:

```python
jwt_secret_key: str
jwt_algorithm: str
access_token_expire_minutes: int
```

Add `User` to `app/models.py`:

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    email: Mapped[str] = mapped_column(String(100), unique=True, index=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(default=True)
```

Add to `app/schemas.py`:

```python
class UserCreate(BaseModel):
    username: str = Field(min_length=3)
    email: str = Field(min_length=5)
    password: str = Field(min_length=6)


class UserResponse(BaseModel):
    id: int
    username: str
    email: str
    is_active: bool

    model_config = {"from_attributes": True}


class Token(BaseModel):
    access_token: str
    token_type: str
```

Create `app/security.py`:

```python
from datetime import datetime, timedelta, timezone

import jwt
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jwt.exceptions import InvalidTokenError
from pwdlib import PasswordHash
from sqlalchemy.orm import Session

from app.config import settings
from app.db import get_db
from app.models import User

password_hash = PasswordHash.recommended()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/login")


def hash_password(password: str):
    return password_hash.hash(password)


def verify_password(plain_password: str, hashed_password: str):
    return password_hash.verify(plain_password, hashed_password)


def create_access_token(data: dict):
    expire = datetime.now(timezone.utc) + timedelta(
        minutes=settings.access_token_expire_minutes
    )

    payload = data.copy()
    payload.update({"exp": expire})

    return jwt.encode(
        payload,
        settings.jwt_secret_key,
        algorithm=settings.jwt_algorithm,
    )


def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db),
):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )

    try:
        payload = jwt.decode(
            token,
            settings.jwt_secret_key,
            algorithms=[settings.jwt_algorithm],
        )
        username = payload.get("sub")

        if username is None:
            raise credentials_exception

    except InvalidTokenError:
        raise credentials_exception

    user = db.query(User).filter(User.username == username).first()

    if user is None:
        raise credentials_exception

    return user
```

Create `app/routers/auth.py`:

```python
from fastapi import APIRouter, Depends, HTTPException
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy.exc import IntegrityError
from sqlalchemy.orm import Session

from app.db import get_db
from app.models import User
from app.schemas import Token, UserCreate, UserResponse
from app.security import create_access_token, get_current_user, hash_password, verify_password

router = APIRouter(prefix="/auth", tags=["auth"])


@router.post("/register", response_model=UserResponse, status_code=201)
def register(user_data: UserCreate, db: Session = Depends(get_db)):
    user = User(
        username=user_data.username.strip(),
        email=user_data.email.strip(),
        hashed_password=hash_password(user_data.password),
    )

    try:
        db.add(user)
        db.commit()
        db.refresh(user)
    except IntegrityError:
        db.rollback()
        raise HTTPException(status_code=400, detail="Username or email already exists")

    return user


@router.post("/login", response_model=Token)
def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: Session = Depends(get_db),
):
    user = db.query(User).filter(User.username == form_data.username).first()

    if user is None or not verify_password(form_data.password, user.hashed_password):
        raise HTTPException(status_code=401, detail="Incorrect username or password")

    access_token = create_access_token(data={"sub": user.username})

    return {"access_token": access_token, "token_type": "bearer"}


@router.get("/me", response_model=UserResponse)
def read_current_user(current_user: User = Depends(get_current_user)):
    return current_user
```

Update `app/main.py`:

```python
from app.routers import auth, categories, expenses

app.include_router(auth.router)
app.include_router(categories.router)
app.include_router(expenses.router)
```

Run Alembic migration:

```powershell
alembic revision --autogenerate -m "add users table"
alembic upgrade head
```

Run API:

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test in this order:

- `POST /auth/register`
- `POST /auth/login`
- Click `Authorize` in Swagger
- Paste username/password in the login popup
- Test `GET /auth/me`

Use register JSON:

```json
{
  "username": "hari",
  "email": "hari@example.com",
  "password": "secret123"
}
```

**Challenge**
Protect `POST /expenses` so only logged-in users can create expenses.

Add this dependency to the route:

```python
current_user: User = Depends(get_current_user)
```

Day 20 is complete when you can explain this flow:

`Register -> hash password -> login -> create JWT -> send JWT in Authorization header -> FastAPI verifies token -> protected route allows access.`
