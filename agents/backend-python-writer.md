# Agent: Backend Python Writer

Escreve codigo backend em Python — endpoints FastAPI/Flask, services, models, schemas.
Usar quando o projeto usa Python no backend.
Qualquer arquivo em `src/`, `app/`, `api/`, `services/`.

## Quando usar este agente

| Usar backend-python-writer | Usar outro agente |
|---|---|
| API REST/GraphQL em Python (FastAPI, Flask, Django) | Backend Node.js (usar backend-node-writer) |
| Scripts e automacoes em Python | Next.js API Routes (usar route-writer) |
| ML/AI endpoints | Frontend React (usar component-writer) |

## Arquitetura padrao (FastAPI)

```
app/
├── main.py                    ← Entry point (FastAPI app)
├── config.py                  ← Settings e env vars
├── database.py                ← Conexao com banco
├── routers/
│   ├── __init__.py
│   ├── auth.py                ← Endpoints de auth
│   └── tasks.py               ← Endpoints de tasks
├── services/
│   ├── __init__.py
│   ├── auth_service.py        ← Logica de negocio auth
│   └── task_service.py        ← Logica de negocio tasks
├── models/
│   ├── __init__.py
│   ├── user.py                ← SQLAlchemy/SQLModel model
│   └── task.py
├── schemas/
│   ├── __init__.py
│   ├── user.py                ← Pydantic schemas (request/response)
│   └── task.py
├── middleware/
│   ├── __init__.py
│   └── auth.py                ← Dependencia de autenticacao
└── utils/
    ├── __init__.py
    └── errors.py              ← Exceptions customizadas
```

## Separacao de responsabilidades

### Router — define endpoints

```python
# app/routers/tasks.py
from fastapi import APIRouter, Depends, Query
from app.schemas.task import TaskCreate, TaskResponse, TaskListResponse
from app.services.task_service import TaskService
from app.middleware.auth import get_current_user
from app.models.user import User

router = APIRouter(prefix="/tasks", tags=["tasks"])

@router.post("/", response_model=TaskResponse, status_code=201)
async def create_task(
    data: TaskCreate,
    current_user: User = Depends(get_current_user),
    service: TaskService = Depends(),
):
    task = await service.create(data=data, user_id=current_user.id)
    return task

@router.get("/", response_model=TaskListResponse)
async def list_tasks(
    page: int = Query(1, ge=1),
    limit: int = Query(20, ge=1, le=100),
    client_id: str | None = None,
    current_user: User = Depends(get_current_user),
    service: TaskService = Depends(),
):
    return await service.list(
        user_id=current_user.id,
        page=page,
        limit=limit,
        client_id=client_id,
    )
```

### Schema — validacao com Pydantic

```python
# app/schemas/task.py
from pydantic import BaseModel, Field
from datetime import datetime
from enum import Enum

class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

class TaskCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    description: str | None = None
    client_id: str
    priority: Priority = Priority.MEDIUM
    deadline: datetime | None = None

class TaskResponse(BaseModel):
    id: str
    title: str
    description: str | None
    priority: Priority
    deadline: datetime | None
    client_id: str
    created_at: datetime

    model_config = {"from_attributes": True}

class TaskListResponse(BaseModel):
    data: list[TaskResponse]
    total: int
```

### Service — logica de negocio

```python
# app/services/task_service.py
from datetime import datetime
from fastapi import Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, func
from app.database import get_session
from app.models.task import Task
from app.schemas.task import TaskCreate

class TaskService:
    def __init__(self, session: AsyncSession = Depends(get_session)):
        self.session = session

    async def create(self, data: TaskCreate, user_id: str) -> Task:
        if data.deadline and data.deadline < datetime.now():
            raise HTTPException(
                status_code=400,
                detail="Deadline deve ser no futuro",
            )

        task = Task(
            **data.model_dump(),
            user_id=user_id,
        )
        self.session.add(task)
        await self.session.commit()
        await self.session.refresh(task)
        return task

    async def list(
        self,
        user_id: str,
        page: int = 1,
        limit: int = 20,
        client_id: str | None = None,
    ):
        query = select(Task).where(Task.user_id == user_id)
        if client_id:
            query = query.where(Task.client_id == client_id)

        total_query = select(func.count()).select_from(query.subquery())
        total = (await self.session.execute(total_query)).scalar() or 0

        items = (
            await self.session.execute(
                query.offset((page - 1) * limit)
                .limit(limit)
                .order_by(Task.created_at.desc())
            )
        ).scalars().all()

        return {"data": items, "total": total}
```

### Model — SQLAlchemy/SQLModel

```python
# app/models/task.py
from sqlalchemy import Column, String, DateTime, Enum, ForeignKey, func
from sqlalchemy.orm import relationship
from app.database import Base
import enum

class Priority(str, enum.Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

class Task(Base):
    __tablename__ = "tasks"

    id = Column(String, primary_key=True, default=generate_cuid)
    title = Column(String, nullable=False)
    description = Column(String, nullable=True)
    priority = Column(Enum(Priority), default=Priority.MEDIUM)
    deadline = Column(DateTime, nullable=True)
    client_id = Column(String, ForeignKey("clients.id", ondelete="CASCADE"))
    user_id = Column(String, ForeignKey("users.id", ondelete="CASCADE"))
    created_at = Column(DateTime, server_default=func.now())
    updated_at = Column(DateTime, server_default=func.now(), onupdate=func.now())

    client = relationship("Client", back_populates="tasks")
    user = relationship("User", back_populates="tasks")
```

### Auth middleware

```python
# app/middleware/auth.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from app.utils.jwt import verify_token
from app.database import get_session

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    session = Depends(get_session),
):
    payload = verify_token(credentials.credentials)
    if not payload:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token invalido ou expirado",
        )
    # buscar user no banco
    user = await session.get(User, payload["sub"])
    if not user:
        raise HTTPException(status_code=404, detail="Usuario nao encontrado")
    return user
```

## Principios

1. **Router → Service → Model** — separacao clara de responsabilidades
2. **Pydantic para validacao** — schemas de request E response
3. **Dependency Injection** — usar `Depends()` do FastAPI para session, auth, services
4. **Async por padrao** — usar `async/await` para I/O
5. **Type hints em tudo** — Python 3.10+ syntax (`str | None` em vez de `Optional[str]`)
6. **Exceptions com HTTP status** — `HTTPException` com status code correto
7. **Alembic para migrations** — nunca alterar schema sem migration

## O que NAO fazer

- Nao colocar logica de negocio no router
- Nao usar `Any` nos types
- Nao fazer queries sem paginacao
- Nao retornar stack traces para o client
- Nao esquecer `await` em operacoes async
- Nao misturar sync e async na mesma cadeia
- Nao hardcodar secrets (usar `config.py` com env vars)

---

## Prompt de Execucao

Quando o /execute delegar a criacao de backend Python, montar o prompt assim:

```
Crie [router/service/model/schema] [NOME] em [PATH].

## Contexto
- Issue: [issue-id]
- Framework: [FastAPI / Flask / Django — de stack.md]
- Descricao: [o que esse modulo faz]

## Camada e responsabilidade
- Router: [endpoints, metodos HTTP]
- Service: [logica de negocio]
- Model: [tabela SQLAlchemy]
- Schema: [Pydantic schemas de request/response]

## Endpoint(s)
| Metodo | Path | Descricao | Auth |
|---|---|---|---|
| [GET/POST/etc.] | [/api/...] | [descricao] | [sim/nao] |

## Schemas Pydantic
- Request: [campos tipados]
- Response: [campos tipados]

## Banco de dados
- ORM: [SQLAlchemy / Django ORM / Tortoise — de stack.md]
- [operacao em qual tabela]
- Async: [sim/nao]

## Dependencies (Depends)
- [db session / auth / service injection]

## Regras de negocio (de .claude/docs/business-rules.md)
- [regra relevante]

## Testes esperados
- Service: [cenarios unitarios com pytest]
- Router: [cenarios com TestClient]
```
