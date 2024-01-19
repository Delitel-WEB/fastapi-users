# SQLAlchemy

**FastAPI Users** предоставляет необходимые инструменты для работы с SQL-базами данных благодаря [SQLAlchemy ORM с поддержкой asyncio](https://docs.sqlalchemy.org/en/14/orm/extensions/asyncio.html).

## Асинхронный драйвер

Для работы с вашей СУБД вам нужно установить соответствующий асинхронный драйвер. Обычными выборами являются:

* Для PostgreSQL: `pip install asyncpg`
* Для SQLite: `pip install aiosqlite`

Примеры `DB_URL`:

* PostgreSQL: `engine = create_engine('postgresql+asyncpg://user:password@host:port/name')`
* SQLite: `engine = create_engine('sqlite+aiosqlite:///name.db')`

Для этого учебного пособия мы будем использовать простую базу данных SQLite.

!!! warning
    При использовании асинхронных сессий убедитесь, что `Session.expire_on_commit` установлен в `False`, как рекомендуется в [документации SQLAlchemy по asyncio](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#asyncio-orm-avoid-lazyloads). Примеры в этой документации уже корректно устанавливают это значение в `False` при использовании фабрики `async_sessionmaker`.

## Создание модели пользователя

Как и для любой модели SQLAlchemy ORM, мы создадим модель `User`.

```py hl_lines="15-16"
--8<-- "docs/src/db_sqlalchemy.py"
```

Как видите, **FastAPI Users** предоставляет базовый класс, который включает основные поля для нашей таблицы `User`. Конечно же, вы можете добавить свои собственные поля, чтобы соответствовать вашим потребностям!

!!! tip "Первичный ключ определен как UUID"
    По умолчанию мы используем UUID в качестве первичного ключа ID для вашего пользователя. Если вы хотите использовать другой тип, например, auto-incremented integer, вы можете использовать `SQLAlchemyBaseUserTable` в качестве базового класса и определить свой собственный столбец `id`.

    ```py
    class User(SQLAlchemyBaseUserTable[int], Base):
        id: Mapped[int] = mapped_column(Integer, primary_key=True)
    ```

    Обратите внимание, что `SQLAlchemyBaseUserTable` ожидает обобщенный тип для определения фактического типа ID, который вы используете.

## Реализация функции для создания таблиц

Теперь мы создадим утилитарную функцию для создания всех определенных таблиц.

```py hl_lines="23-25"
--8<-- "docs/src/db_sqlalchemy.py"
```

Эту функцию можно вызвать, например, во время инициализации вашего приложения FastAPI.

!!! warning
    В производственной среде настоятельно рекомендуется настроить систему миграции для обновления ваших схем SQL. См. [Alembic](https://alembic.sqlalchemy.org/en/latest/).

## Создание зависимости адаптера базы данных

Адаптер базы данных **FastAPI Users** создает связь между конфигурацией вашей базы данных и логикой пользователей. Его следует создавать зависимостью FastAPI.

```py hl_lines="28-34"
--8<-- "docs/src/db_sqlalchemy.py"
```

Обратите внимание, что мы сначала определяем зависимость `get_async_session`, возвращающую нам свежую сессию SQLAlchemy для взаимодействия с базой данных.

Затем она используется в зависимости `get_user_db`, чтобы сгенерировать наш адаптер. Обратите внимание, что мы передаем ему две вещи:

* Экземпляр `session`, который мы только что внедрили.
* Класс `User`, который является фактической моделью SQLAlchemy.