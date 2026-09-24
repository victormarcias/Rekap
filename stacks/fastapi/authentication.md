# Authentication

Concrete implementation of registration + login with Argon2 and JWT. The underlying concepts (why hash, what a salt is, stateless JWT) are in [Authentication and Security — general concepts](../../backend/authentication.md).

## 1. Hashing passwords with Argon2 (`passlib`)

`passlib` generates the salt automatically and embeds it in the resulting string — no need to manage it by hand.

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["argon2"], deprecated="auto")

hashed = pwd_context.hash("my-super-secret-password")
pwd_context.verify("my-super-secret-password", hashed)  # True
pwd_context.verify("wrong-password", hashed)          # False
```

## 2. Registration: never persist or return the plaintext password

The hash is computed before saving, and the endpoint's response never includes the original password or the hash — leaking the hash isn't as serious as the plaintext, but it's still information an attacker could use to try to crack it offline.

```python
class RegisterBody(BaseModel):
    email: str
    password: str  # never persisted as-is

@app.post("/register")
async def register(body: RegisterBody, db: AsyncSession = Depends(get_db)):
    hashed_password = pwd_context.hash(body.password)
    user = User(email=body.email, hashed_password=hashed_password)
    db.add(user)
    await db.commit()
    return {"id": user.id, "email": user.email}  # never the hash in the response
```

## 3. Login and JWT generation

The same error message for "user doesn't exist" and "wrong password" prevents someone from using the endpoint to discover which emails are registered (*user enumeration*) — giving different messages is a subtle but real information leak.

```python
from datetime import datetime, timedelta, timezone
from jose import jwt
from fastapi.security import OAuth2PasswordRequestForm

def create_access_token(user_id: int) -> str:
    expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    payload = {"sub": str(user_id), "exp": expire}
    return jwt.encode(payload, settings.SECRET_KEY, algorithm=settings.ALGORITHM)

@app.post("/login")
async def login(form_data: OAuth2PasswordRequestForm = Depends(), db: AsyncSession = Depends(get_db)):
    user = await get_user_by_email(db, form_data.username)
    # ✅ same error whether the user doesn't exist or the password is wrong
    if not user or not pwd_context.verify(form_data.password, user.hashed_password):
        raise HTTPException(status_code=401, detail="Invalid credentials")
    token = create_access_token(user.id)
    return {"access_token": token, "token_type": "bearer"}
```

## 4. `pydantic-settings` for configuration

A `SECRET_KEY` hardcoded in the code ends up, sooner or later, committed to git. Reading it from environment variables / `.env` (which goes in `.gitignore`) keeps the secret from traveling with the source code.

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 15

    class Config:
        env_file = ".env"

settings = Settings()  # reads from environment variables / .env — never hardcoded in the code
```

## 5. Protecting routes with `OAuth2PasswordBearer` + `Depends`

`OAuth2PasswordBearer` automatically extracts the `Authorization: Bearer <token>` header, and makes the "Authorize" button appear only in the `/docs` documentation (see [Automatic documentation](microservice-endpoints.md)) — any route that depends on `get_current_user` ends up protected without repeating the validation logic in every endpoint.

```python
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="login")

async def get_current_user(token: str = Depends(oauth2_scheme), db: AsyncSession = Depends(get_db)):
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        user_id = payload.get("sub")
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
    user = await get_user_by_id(db, int(user_id))
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    return user

@app.get("/me")
async def read_current_user(current_user: User = Depends(get_current_user)):
    return {"email": current_user.email}
```

## 6. What to never log

A debug log with sensitive data can end up sitting for months in a centralized logging system, exposed to anyone with access to those logs — far more people than should ever be able to see a password.

```python
# ❌ even in debug, this can end up persisted and accessible to third parties
logger.debug(f"Login attempt: {form_data.username} / {form_data.password}")

# ✅ the event is logged with no sensitive data at all
logger.info(f"Login attempt for user_id={user.id if user else 'unknown'}")
```

---
Related: [Authentication and Security — general concepts](../../backend/authentication.md), [Microservice Endpoints](microservice-endpoints.md) (`Depends`, exception handlers), [Sync vs Async in FastAPI](sync-vs-async.md).
