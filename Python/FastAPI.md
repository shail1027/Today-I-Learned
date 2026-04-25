# FastAPI란


## FastAPI란?

FastAPI는 Python 3.7+ 기반의 고성능 비동기 웹 프레임워크로, 내부적으로 Starlette(ASGI 기반 웹 레이어)와 Pydantic(타입 힌트 기반 데이터 검증 및 직렬화)이라는 두 라이브러리 위에 세워진다. ASGI(Asynchronous Server Gateway Interface) 아키텍처를 채택한 덕분에 단일 이벤트 루프에서 수많은 요청을 비동기로 처리할 수 있어, 기존 WSGI 기반의 Flask나 Django에 비해 Node.js, Go에 근접하는 처리 성능을 보인다. 또한 Python의 타입 힌트를 적극적으로 활용하여 요청 데이터의 자동 검증, IDE 자동완성, 그리고 별도 설정 없이 Swagger UI(/docs)와 ReDoc(/redoc)을 자동으로 생성해주는 자동 문서화 기능을 갖추고 있다.

### 왜 FastAPI를 쓰는가?

| 특징 | 설명 |
|---|---|
| 빠른 속도 | NodeJS, Go에 근접하는 성능 (비동기 ASGI 덕분) |
| 자동 검증 | 요청 데이터를 Pydantic이 자동으로 검증, 잘못된 데이터는 422 반환 |
| 자동 문서화 | 코드 작성만으로 Swagger UI, ReDoc 자동 생성 |
| 타입 안정성 | Python 타입 힌트 기반 → IDE 자동완성, 런타임 오류 감소 |
| 직관적 DI | `Depends()`로 의존성 주입을 간결하게 표현 |

---

## 파일 구조

```
project/
├── app/
│   ├── main.py                  # FastAPI 앱 생성, 라우터/미들웨어 등록
│   ├── dependencies.py          # 공통 의존성 (DB 세션, 현재 유저 등)
│   │
│   ├── routers/                 # 도메인별 라우터 분리
│   │   ├── __init__.py
│   │   ├── users.py             # GET/POST/PUT/DELETE /users
│   │   ├── items.py
│   │   └── auth.py              # 로그인, 토큰 발급
│   │
│   ├── models/                  # SQLAlchemy ORM 모델 (DB 테이블 정의)
│   │   ├── __init__.py
│   │   ├── base.py              # declarative_base() 정의
│   │   └── user.py
│   │
│   ├── schemas/                 # Pydantic 스키마 (API 입출력 형식)
│   │   ├── __init__.py
│   │   └── user.py              # UserCreate, UserUpdate, UserResponse 등
│   │
│   ├── crud/                    # DB 조작 함수 (비즈니스 로직과 분리)
│   │   ├── __init__.py
│   │   └── user.py              # get_user, create_user, delete_user 등
│   │
│   ├── core/
│   │   ├── config.py            # 환경변수 로드 (BaseSettings)
│   │   ├── security.py          # JWT 생성/검증, 비밀번호 해싱
│   │   └── database.py          # DB 엔진, 세션 팩토리 생성
│   │
│   └── middlewares/
│       └── logging.py           # 커스텀 미들웨어
│
├── alembic/                     # DB 마이그레이션
│   └── versions/
├── tests/
│   ├── conftest.py              # pytest fixture
│   └── test_users.py
├── .env                         # 환경 변수 (SECRET_KEY, DATABASE_URL 등)
├── alembic.ini
└── requirements.txt
```

### models vs schemas — 왜 분리하는가?

```
models/user.py          ->  DB 테이블 구조 (SQLAlchemy)
schemas/user.py         ->  API 입출력 형식 (Pydantic)
```

ORM 모델을 그대로 API 응답으로 쓰면:
- 비밀번호 해시 같은 **민감 필드가 노출**될 수 있음
- DB 구조 변경이 API 스펙에 즉시 영향을 줌 -> **결합도 증가**

분리하면:
- 응답에 보여줄 필드를 **명시적으로 제어** 가능
- DB 구조와 API 스펙을 독립적으로 변경 가능


## 동작 원리

### 1. 전체 요청 흐름

```
[클라이언트]
    │  HTTP Request
    v
[Uvicorn - ASGI 서버]
    │  ASGI 프로토콜로 변환
    v
[Starlette - 웹 레이어]
    │
    ├─> [미들웨어 체인]  ← 요청이 핸들러 도달 전 모두 통과
    │       CORS / 로깅 / 인증 / 압축 ...
    │
    ├─> [라우터 매칭]
    │       URL + HTTP Method 기준으로 핸들러 탐색
    │
    ├─> [의존성 트리 해소]
    │       Depends() 체인을 재귀적으로 실행
    │       (DB 세션 열기, 토큰 검증 등)
    │
    ├─> [Pydantic - 요청 데이터 검증]
    │       Body, Query, Path 파라미터 파싱 및 타입 검증
    │       실패 시 → 422 Unprocessable Entity 자동 반환
    │
    ├─> [핸들러 함수 실행]
    │       async def → 이벤트 루프에서 실행
    │       def       → threadpool executor에서 실행
    │
    └─> [Pydantic - 응답 직렬화]
            response_model 기준으로 필드 필터링 후 JSON 변환

    │  HTTP Response
    v
[클라이언트]
```

### 2. ASGI와 Uvicorn

- **WSGI**(Flask, Django 구버전): 요청 하나당 스레드 하나 -> 동시 처리 한계
- **ASGI**: 단일 이벤트 루프에서 수천 개의 요청을 비동기 처리 가능

```bash
# Uvicorn이 FastAPI 앱을 ASGI 앱으로 실행
uvicorn app.main:app --reload
# app.main  -> app/main.py 파일
# :app      -> 해당 파일 안의 FastAPI() 인스턴스 변수명
```

### 3. async def vs def

```python
# async def — 비동기 I/O (DB, HTTP 외부 호출 등)
@app.get("/users")
async def get_users():
    users = await db.fetch_all("SELECT * FROM users")  # 논블로킹
    return users

# def — CPU 바운드 작업 (이미지 처리, 수학 연산 등)
@app.post("/resize")
def resize_image(file: UploadFile):
    return process_image(file)  # threadpool에서 실행됨
```

> `async def` 안에서 동기 블로킹 함수(`time.sleep`, `requests` 등)를 쓰면  
> 이벤트 루프 전체가 멈춰 성능이 급격히 저하됨.  
> 이 경우 `asyncio.run_in_executor()` 또는 `anyio.to_thread.run_sync()` 사용


## 핵심 개념

### 1. Pydantic 스키마

```python
from pydantic import BaseModel, EmailStr, Field, field_validator
from typing import Optional
from datetime import datetime

# 요청용 스키마 (클라이언트 -> 서버)
class UserCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=50, description="사용자 이름")
    email: EmailStr                          # 이메일 형식 자동 검증
    age: int = Field(..., ge=0, le=150)      # 0 이상 150 이하
    password: str = Field(..., min_length=8)

    @field_validator("name")
    @classmethod
    def name_must_not_contain_space(cls, v: str) -> str:
        if " " in v:
            raise ValueError("이름에 공백이 포함될 수 없습니다")
        return v.strip()

# 업데이트용 (모든 필드 선택적)
class UserUpdate(BaseModel):
    name: Optional[str] = None
    email: Optional[EmailStr] = None

# 응답용 스키마 (서버 -> 클라이언트)
class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    created_at: datetime
    # password는 포함하지 않음 → 자동으로 응답에서 제외됨

    model_config = {"from_attributes": True}  # Pydantic v2: ORM 객체 -> 스키마 변환
```

#### 스키마 상속으로 중복 제거

```python
class UserBase(BaseModel):
    name: str
    email: EmailStr

class UserCreate(UserBase):       # UserBase + password
    password: str

class UserResponse(UserBase):     # UserBase + id, created_at
    id: int
    created_at: datetime
    model_config = {"from_attributes": True}
```

### 2. 의존성 주입 (Dependency Injection)

#### 기본 패턴 — DB 세션

```python
# core/database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "postgresql://user:pass@localhost/db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

# dependencies.py
def get_db():
    db = SessionLocal()
    try:
        yield db          # 핸들러 실행 중 DB 세션 제공
    finally:
        db.close()        # 핸들러 종료 후 반드시 세션 닫힘 (yield 덕분에 보장)
```

#### 의존성 체인 (중첩 Depends)

```python
# 토큰 검증 → 유저 조회 → 관리자 권한 확인까지 체인
def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    payload = verify_token(token)
    user = db.query(User).filter(User.id == payload["sub"]).first()
    if not user:
        raise HTTPException(status_code=401, detail="인증 실패")
    return user

def get_admin_user(
    current_user: User = Depends(get_current_user)
) -> User:
    if not current_user.is_admin:
        raise HTTPException(status_code=403, detail="권한 없음")
    return current_user

@app.delete("/users/{user_id}")
def delete_user(
    user_id: int,
    admin: User = Depends(get_admin_user)   # 관리자만 접근 가능
):
    ...
```

#### 클래스 기반 의존성 (재사용 가능한 파라미터 묶음)

```python
class PaginationParams:
    def __init__(self, page: int = 1, size: int = 20):
        self.offset = (page - 1) * size
        self.limit = size

@app.get("/items")
def list_items(pagination: PaginationParams = Depends()):
    return db.query(Item).offset(pagination.offset).limit(pagination.limit).all()
```

### 3. 라우터 분리 (APIRouter)

```python
# routers/users.py
from fastapi import APIRouter, Depends, HTTPException, status

router = APIRouter(
    prefix="/users",
    tags=["Users"],
    dependencies=[Depends(get_current_user)],  # 이 라우터 전체에 인증 적용
    responses={404: {"description": "Not found"}},
)

@router.get("/", response_model=list[UserResponse])
def list_users(db: Session = Depends(get_db)):
    return db.query(User).all()

@router.get("/{user_id}", response_model=UserResponse)
def get_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="유저를 찾을 수 없습니다")
    return user

@router.post("/", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
def create_user(body: UserCreate, db: Session = Depends(get_db)):
    user = User(**body.model_dump())
    db.add(user)
    db.commit()
    db.refresh(user)
    return user
```

```python
# main.py
from fastapi import FastAPI
from app.routers import users, items, auth

app = FastAPI(title="My API", version="1.0.0")

app.include_router(auth.router)
app.include_router(users.router)
app.include_router(items.router, prefix="/api/v1")  # 추가 prefix 붙이기도 가능
```

### 4. 경로 파라미터 & 쿼리 파라미터

```python
from fastapi import Path, Query
from typing import Literal

@app.get("/users/{user_id}/posts")
def get_user_posts(
    user_id: int = Path(..., ge=1, description="유저 ID"),           # 경로 파라미터
    page: int = Query(1, ge=1, description="페이지 번호"),            # 쿼리 파라미터
    keyword: str | None = Query(None, max_length=100),               # 선택적 쿼리
    sort: Literal["asc", "desc"] = Query("desc"),
):
    ...
# 요청 예: GET /users/42/posts?page=2&keyword=fastapi&sort=asc
```

### 5. 예외 처리

```python
from fastapi import HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

# 기본 HTTPException
raise HTTPException(
    status_code=404,
    detail="리소스를 찾을 수 없습니다",
    headers={"X-Error": "NotFound"},
)

# 전역 예외 핸들러 — 422 응답 커스터마이징
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return JSONResponse(
        status_code=422,
        content={"message": "입력값이 올바르지 않습니다", "detail": exc.errors()},
    )

# 커스텀 예외 클래스
class UserNotFoundException(Exception):
    def __init__(self, user_id: int):
        self.user_id = user_id

@app.exception_handler(UserNotFoundException)
async def user_not_found_handler(request, exc):
    return JSONResponse(
        status_code=404,
        content={"message": f"유저 {exc.user_id}를 찾을 수 없습니다"},
    )
```

---

## 미들웨어 심화

### 실행 순서

```
요청  ->  미들웨어 A (전처리)
     ->  미들웨어 B (전처리)
     ->  핸들러 실행
     <-  미들웨어 B (후처리)
     <-  미들웨어 A (후처리)
응답
```

### 커스텀 미들웨어 작성

```python
import time
from fastapi import Request

@app.middleware("http")
async def logging_middleware(request: Request, call_next):
    start = time.time()

    # 전처리
    print(f"[요청] {request.method} {request.url}")

    response = await call_next(request)  # 실제 핸들러 실행

    # 후처리
    process_time = time.time() - start
    response.headers["X-Process-Time"] = str(process_time)
    print(f"[응답] {response.status_code} - {process_time:.3f}s")

    return response
```

### CORS 미들웨어

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://myapp.com", "http://localhost:3000"],
    allow_credentials=True,      # 쿠키 허용
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)
```

---

## JWT 인증 흐름

```python
# core/security.py
from datetime import datetime, timedelta
from jose import jwt, JWTError
from passlib.context import CryptContext

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"])

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

def create_access_token(data: dict) -> str:
    payload = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    payload.update({"exp": expire})
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except JWTError:
        raise HTTPException(status_code=401, detail="유효하지 않은 토큰")
```

```python
# routers/auth.py
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/login")

@router.post("/login")
def login(form: OAuth2PasswordRequestForm = Depends(), db: Session = Depends(get_db)):
    user = db.query(User).filter(User.email == form.username).first()
    if not user or not verify_password(form.password, user.password_hash):
        raise HTTPException(status_code=401, detail="이메일 또는 비밀번호가 올바르지 않습니다")

    token = create_access_token({"sub": str(user.id)})
    return {"access_token": token, "token_type": "bearer"}
```

---

## SQLAlchemy 연동 패턴

```python
# models/user.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime, func
from app.models.base import Base

class User(Base):
    __tablename__ = "users"

    id         = Column(Integer, primary_key=True, index=True)
    name       = Column(String(50), nullable=False)
    email      = Column(String(100), unique=True, index=True, nullable=False)
    password_hash = Column(String, nullable=False)
    is_admin   = Column(Boolean, default=False)
    created_at = Column(DateTime, server_default=func.now())
    updated_at = Column(DateTime, onupdate=func.now())
```

```python
# crud/user.py
def get_user(db: Session, user_id: int) -> User | None:
    return db.query(User).filter(User.id == user_id).first()

def create_user(db: Session, data: UserCreate) -> User:
    user = User(
        name=data.name,
        email=data.email,
        password_hash=hash_password(data.password),
    )
    db.add(user)
    db.commit()
    db.refresh(user)    # DB에서 최신 상태 (id, created_at 등) 다시 로드
    return user

def update_user(db: Session, user: User, data: dict) -> User:
    for key, value in data.items():
        setattr(user, key, value)
    db.commit()
    db.refresh(user)
    return user
```

---

## 자동 문서화 상세

```python
app = FastAPI(
    title="My Service API",
    description="## 마이 서비스 API\n유저 및 아이템 관리 API입니다.",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc",
)

@router.post(
    "/users",
    response_model=UserResponse,
    status_code=201,
    summary="유저 생성",
    description="새로운 유저를 생성합니다. 이메일은 중복 불가합니다.",
    response_description="생성된 유저 정보",
    tags=["Users"],
    responses={
        409: {"description": "이미 존재하는 이메일"},
        422: {"description": "입력값 오류"},
    },
)
def create_user(body: UserCreate):
    ...
```

| URL | 설명 |
|---|---|
| `/docs` | Swagger UI — 인터랙티브 API 테스트 가능 |
| `/redoc` | ReDoc — 읽기 좋은 문서 형식 |
| `/openapi.json` | OpenAPI 스펙 JSON 원본 |

---

## 배포 & 실행

### 개발 환경

```bash
pip install fastapi "uvicorn[standard]" sqlalchemy "pydantic[email]" python-jose passlib

uvicorn app.main:app --reload --port 8000
```

### 프로덕션 (Gunicorn + Uvicorn Workers)

```bash
pip install gunicorn

gunicorn app.main:app \
  -w 4 \                                    # 워커 수: CPU 코어 × 2 + 1 권장
  -k uvicorn.workers.UvicornWorker \
  --bind 0.0.0.0:8000 \
  --timeout 120
```

### Docker

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 테스트

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.dependencies import get_db
from app.models.base import Base

engine = create_engine("sqlite:///./test.db", connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(bind=engine)

@pytest.fixture
def client():
    Base.metadata.create_all(bind=engine)

    def override_get_db():
        db = TestingSessionLocal()
        try:
            yield db
        finally:
            db.close()

    app.dependency_overrides[get_db] = override_get_db  # 실제 DB 대신 테스트 DB로 교체
    yield TestClient(app)
    Base.metadata.drop_all(bind=engine)

# tests/test_users.py
def test_create_user(client):
    response = client.post("/users", json={
        "name": "홍길동",
        "email": "hong@example.com",
        "age": 30,
        "password": "securepass"
    })
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "hong@example.com"
    assert "password" not in data   # 응답에 비밀번호 없음 확인
```
