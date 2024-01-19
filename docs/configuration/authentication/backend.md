# Создание бэкенда

Как мы уже сказали, бэкенд - это комбинация транспорта и стратегии. Таким образом, вы можете создать полноценную стратегию, полностью соответствующую вашим потребностям.

Для этого вы должны использовать класс `AuthenticationBackend`.

```py
from fastapi_users.authentication import AuthenticationBackend, BearerTransport, JWTStrategy

SECRET = "SECRET"

bearer_transport = BearerTransport(tokenUrl="auth/jwt/login")

def get_jwt_strategy() -> JWTStrategy:
    return JWTStrategy(secret=SECRET, lifetime_seconds=3600)

auth_backend = AuthenticationBackend(
    name="jwt",
    transport=bearer_transport,
    get_strategy=get_jwt_strategy,
)
```

Как видите, создание экземпляра довольно просто. Он принимает следующие аргументы:

* `name` (`str`): Имя бэкенда. Каждый бэкенд должен иметь уникальное имя.
* `transport` (`Transport`): Экземпляр класса `Transport`.
* `get_strategy` (`Callable[..., Strategy]`): Вызываемый зависимый, возвращающий экземпляр класса `Strategy`.

## Следующие шаги

У вас может быть столько бэкендов аутентификации, сколько вам нужно. Затем вы должны передать эти бэкенды в ваш экземпляр `FastAPIUsers` и создать маршрутизатор аутентификации для каждого из них.