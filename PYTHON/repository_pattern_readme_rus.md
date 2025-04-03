# Чистая архитектура доступа к данным: PostgreSQL и MongoDB

## 🧠 Зачем всё это нужно

Когда вы начинаете писать проект, вы часто напрямую работаете с базой данных, например, с PostgreSQL через SQLAlchemy. Всё удобно... пока вы не захотите:

- перейти на другую СУБД (например, MongoDB);
- писать тесты без подключения к реальной БД;
- разделить бизнес-логику от работы с хранилищем.

Если вы используете чистую архитектуру и внедряете **интерфейсы**, вы можете это всё легко реализовать.

---

## 📌 Что мы хотим получить

- **Один интерфейс** (описание того, какие методы нужны).
- **Несколько реализаций** (PostgreSQL, MongoDB).
- **Один сервис**, который работает с интерфейсом, не зная, какая конкретная БД под капотом.
- **Гибкость**: сменить БД — просто заменить реализацию.

---

## 📁 Структура проекта

```
project/
├── domain/
│   └── repositories/
│       └── archive_poll_repo.py        # Интерфейс репозитория
├── infrastructure/
│   ├── postgresql/
│   │   └── archive_poll_repo.py        # Реализация под PostgreSQL
│   └── mongo/
│       └── archive_poll_repo.py        # Реализация под MongoDB
└── services/
    └── archive_poll_service.py         # Сервис / use-case слой
```

---

## 1. Интерфейс репозитория

Интерфейс определяет **что нужно уметь делать**, но **не говорит, как именно**.

```python
# domain/repositories/archive_poll_repo.py

from abc import ABC, abstractmethod

class AbstractArchivePollRepository(ABC):
    @abstractmethod
    async def get_paginated(self, limit: int, offset: int) -> list[dict]:
        ...
```

---

## 2. Реализация под PostgreSQL

```python
# infrastructure/postgresql/archive_poll_repo.py

from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from domain.repositories.archive_poll_repo import AbstractArchivePollRepository
from .models import ArchivedPoll

class PgArchivePollRepository(AbstractArchivePollRepository):
    def __init__(self, session: AsyncSession):
        self.session = session

    async def get_paginated(self, limit: int, offset: int) -> list[dict]:
        result = await self.session.execute(
            select(ArchivedPoll).order_by(ArchivedPoll.created_at).offset(offset).limit(limit)
        )
        return [poll.to_dict() for poll in result.scalars().unique().all()]
```

---

## 3. Реализация под MongoDB

```python
# infrastructure/mongo/archive_poll_repo.py

from motor.motor_asyncio import AsyncIOMotorCollection
from domain.repositories.archive_poll_repo import AbstractArchivePollRepository

class MongoArchivePollRepository(AbstractArchivePollRepository):
    def __init__(self, collection: AsyncIOMotorCollection):
        self.collection = collection

    async def get_paginated(self, limit: int, offset: int) -> list[dict]:
        cursor = self.collection.find().sort("created_at", 1).skip(offset).limit(limit)
        return [doc async for doc in cursor]
```

---

## 4. Сервис (use-case слой)

Сервис работает **только с интерфейсом** и не знает, на чём он реализован — SQL или Mongo.

```python
# services/archive_poll_service.py

from domain.repositories.archive_poll_repo import AbstractArchivePollRepository

class ArchivePollService:
    def __init__(self, repo: AbstractArchivePollRepository):
        self.repo = repo

    async def fetch_paginated(self, limit: int, offset: int) -> list[dict]:
        return await self.repo.get_paginated(limit, offset)
```

---

## 5. Как использовать с PostgreSQL

```python
from sqlalchemy.ext.asyncio import AsyncSession
from infrastructure.postgresql.archive_poll_repo import PgArchivePollRepository
from services.archive_poll_service import ArchivePollService

async with AsyncSession(engine) as session:
    repo = PgArchivePollRepository(session)
    service = ArchivePollService(repo)

    async with session.begin():
        result = await service.fetch_paginated(limit=10, offset=0)
```

---

## 6. Как использовать с MongoDB

```python
from motor.motor_asyncio import AsyncIOMotorClient
from infrastructure.mongo.archive_poll_repo import MongoArchivePollRepository
from services.archive_poll_service import ArchivePollService

client = AsyncIOMotorClient("mongodb://localhost:27017")
collection = client["mydb"]["archived_polls"]

repo = MongoArchivePollRepository(collection)
service = ArchivePollService(repo)

result = await service.fetch_paginated(limit=10, offset=0)
```

---

---

## ✅ Плюсы подхода

- 💡 **Гибкость** — можно легко заменить БД.
- 🧪 **Тестируемость** — можно писать юнит-тесты без БД.
- 🧱 **Модульность** — каждая часть отвечает только за своё.
- ♻️ **Повторное использование** — можно использовать одни и те же use-case'ы с разными хранилищами.

---
