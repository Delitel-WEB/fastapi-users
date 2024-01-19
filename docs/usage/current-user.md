# Получение текущего пользователя

**FastAPI Users** предоставляет вызываемую зависимость для легкого внедрения аутентифицированного пользователя в ваши маршруты. Они доступны из вашего экземпляра `FastAPIUsers`.

!!! Совет
    Для получения дополнительной информации о том, как выполнить аутентифицированный запрос к вашему API, ознакомьтесь с документацией вашего [метода аутентификации](../configuration/authentication/index.md).

## `current_user`

Возвращает вызываемую зависимость для получения текущего аутентифицированного пользователя, передавая следующие параметры:

* `optional`: Если `True`, возвращается `None`, если нет аутентифицированного пользователя или если он не проходит другие требования. В противном случае генерируется `401 Unauthorized`. По умолчанию `False`.
* `active`: Если `True`, генерируется `401 Unauthorized`, если аутентифицированный пользователь неактивен. По умолчанию `False`.
* `verified`: Если `True`, генерируется `403 Forbidden`, если аутентифицированный пользователь не подтвержден. По умолчанию `False`.
* `superuser`: Если `True`, генерируется `403 Forbidden`, если аутентифицированный пользователь не является суперпользователем. По умолчанию `False`.
* `get_enabled_backends`: Опциональная вызываемая зависимость, возвращающая список включенных бэкендов аутентификации. Полезно, если вы хотите динамически включать некоторые бэкенды аутентификации на основе внешней логики, например, конфигурации в базе данных. По умолчанию включаются все указанные бэкенды аутентификации. *Однако обратите внимание, что каждый бэкенд будет отображаться в документации OpenAPI, так как FastAPI разрешает это статически.*

!!! Совет "Создайте ее один раз и используйте повторно"
    Эта функция - **фабрика**, функция, возвращающая другую функцию 🤯

    Именно эта возвращенная функция будет вызываемой зависимостью, которую FastAPI вызывает в ваших маршрутах API.

    Чтобы избежать необходимости создавать ее на каждом маршруте и избежать проблем при модульном тестировании, **настоятельно рекомендуется** присвоить результат переменной и использовать его по желанию в ваших маршрутах. Приведенные ниже примеры демонстрируют этот подход.

## Примеры

### Получение текущего пользователя (**активного или нет**)

```py
current_user = fastapi_users.current_user()

@app.get("/protected-route")
def protected_route(user: User = Depends(current_user)):
    return f"Привет, {user.email}"
```

### Получение текущего **активного** пользователя

```py
current_active_user = fastapi_users.current_user(active=True)

@app.get("/protected-route")
def protected_route(user: User = Depends(current_active_user)):
    return f"Привет, {user.email}"
```

### Получение текущего **активного** и **подтвержденного** пользователя

```py
current_active_verified_user = fastapi_users.current_user(active=True, verified=True)

@app.get("/protected-route")
def protected_route(user: User = Depends(current_active_verified_user)):
    return f"Привет, {user.email}"
```

### Получение текущего активного **суперпользователя**

```py
current_superuser = fastapi_users.current_user(active=True, superuser=True)

@app.get("/protected-route")
def protected_route(user: User = Depends(current_superuser)):
    return f"Привет, {user.email}"
```

### Динамическое включение бэкендов аутентификации

!!! Предупреждение
    Это продвинутая функция для случаев, когда у вас есть несколько бэкендов аутентификации, которые включаются условно. В большинстве случаев вам не потребуется эта опция.

```py
from fastapi import Request
from fastapi_users.authentication import AuthenticationBackend, BearerTransport, CookieTransport, JWTStrategy

SECRET = "SECRET"

bearer_transport = BearerTransport(tokenUrl="auth/jwt/login")
cookie_transport = CookieTransport(cookie_max_age=3600)

def get_jwt_strategy() -> JWTStrategy:
    return JWTStrategy(secret=SECRET, lifetime_seconds=3600)

jwt_backend = AuthenticationBackend(
    name="jwt",
    transport=bearer_transport,
    get_strategy=get_jwt_strategy,
)
cookie_backend = AuthenticationBackend(
    name="jwt",
    transport=cookie_transport,
    get_strategy=get_jwt_strategy,
)

async def get_enabled_backends(request: Request):
    """Возвращает включенные зависимости в соответствии с пользовательской логикой."""
    if request.url.path == "/protected-route-only-jwt":
        return [jwt_backend]
    else:
        return [cookie_backend, jwt_backend]


current_active_user = fastapi_users.current_user(active=True, get_enabled_backends=get_enabled_backends)


@app.get("/protected-route")
def protected_route(user: User = Depends(current_active_user)):
    return f"Привет, {user.email}. Вы аутентифицированы с помощью cookie или JWT."


@app.get("/protected-route-only-jwt")
def protected_route(user: User = Depends(current_active_user)):
    return f"Привет, {user.email}. Вы аутентифицированы с помощью JWT."
```

## В операции пути

Если вам не нужен пользователь в логике маршрута, вы можете использовать этот синтаксис:

```py
@app.get("/protected-route", dependencies=[Depends(current_superuser)])
def protected_route():
    return "Привет

, какой-то пользователь."
```

Вы можете узнать больше об этом [в документации FastAPI](https://fastapi.tiangolo.com/tutorial/dependencies/dependencies-in-path-operation-decorators/).