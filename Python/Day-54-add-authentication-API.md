Regular `Day 54`: add authentication to the real portfolio backend.

Today you add:

- `POST /auth/register`
- `POST /auth/login`
- `GET /auth/me`
- Password hashing with `pwdlib`
- JWT tokens with `PyJWT`

FastAPI’s current security guide uses `OAuth2PasswordRequestForm`, `OAuth2PasswordBearer`, `PyJWT`, and `pwdlib`. Login uses form data, so `python-multipart` must be installed. Sources: [FastAPI JWT security](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/), [FastAPI security reference](https://fastapi.tiangolo.com/reference/security/), [FastAPI form data](https://fastapi.tiangolo.com/tutorial/request-forms/).

**Start**

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
python -m pip install pyjwt "pwdlib[argon2]" python-multipart
```

Generate JWT secret:

```powershell
python -c "import secrets; print(secrets.token_hex(32))"
```

Put that value in `.env`:

```env
JWT_SECRET_KEY=paste_generated_secret_here
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Create/update `app/schemas.py`:

```python
from pydantic import BaseModel, EmailStr, Field


class UserCreate(BaseModel):
    username: str = Field(min_length=3, max_length=50)
    email: EmailStr
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

Install email validator:

```powershell
python -m pip install pydantic[email]
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
    expires_at = datetime.now(timezone.utc) + timedelta(
        minutes=settings.access_token_expire_minutes
    )
    payload = data.copy()
    payload.update({"exp": expires_at})
    return jwt.encode(payload, settings.jwt_secret_key, algorithm=settings.jwt_algorithm)


def get_current_user(token: str = Depends(oauth2_scheme), db: Session = Depends(get_db)):
    error = HTTPException(
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
            raise error
    except InvalidTokenError:
        raise error

    user = db.query(User).filter(User.username == username).first()
    if user is None:
        raise error

    return user
```

Create folder/file:

```text
app/routers/auth.py
```

```python
from fastapi import APIRouter, Depends, HTTPException, status
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
def login(form_data: OAuth2PasswordRequestForm = Depends(), db: Session = Depends(get_db)):
    user = db.query(User).filter(User.username == form_data.username).first()

    if user is None or not verify_password(form_data.password, user.hashed_password):
        raise HTTPException(status_code=401, detail="Incorrect username or password")

    token = create_access_token({"sub": user.username})
    return {"access_token": token, "token_type": "bearer"}


@router.get("/me", response_model=UserResponse)
def read_me(current_user: User = Depends(get_current_user)):
    return current_user
```

Update `app/main.py`:

```python
from fastapi import FastAPI

from app.db import check_database_connection
from app.routers import auth

app = FastAPI(title="Expense Manager SaaS API")

app.include_router(auth.router)
```

Keep your existing `/health` and `/health/db` routes below that.

Run:

```powershell
uvicorn app.main:app --reload
```

Test in Swagger:

1. `POST /auth/register`
2. `POST /auth/login`
3. Click `Authorize`
4. Login with username/password
5. `GET /auth/me`

Login is form data, not JSON.

Commit:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"
git add .
git commit -m "Day 54 add authentication API"
```

Day 54 is complete when `/auth/register`, `/auth/login`, and `/auth/me` work correctly.
