# JWT

[JSON Web Token (JWT)](https://jwt.io/introduction) - это стандарт интернета для создания токенов доступа на основе JSON. Их не нужно хранить в базе данных: данные находятся внутри токена и криптографически подписаны.

## Конфигурация

```py
from fastapi_users.authentication import JWTStrategy

SECRET = "SECRET"

def get_jwt_strategy() -> JWTStrategy:
    return JWTStrategy(secret=SECRET, lifetime_seconds=3600)
```

Как видите, создание экземпляра довольно просто. Он принимает следующие аргументы:

- `secret` (`Union[str, pydantic.SecretStr]`): Постоянный секрет, используемый для кодирования токена. **Используйте надежную фразу и храните ее в безопасности.**
- `lifetime_seconds` (`Optional[int]`): Срок действия токена в секундах. Может быть установлен в `None`, но в этом случае токен будет действителен **вечно**; что может вызвать серьезные проблемы безопасности.
- `token_audience` (`Optional[List[str]]`): Список допустимых аудиторий для токена JWT. По умолчанию `["fastapi-users:auth"]`.
- `algorithm` (`Optional[str]`): Алгоритм шифрования JWT. См. [RFC 7519, раздел 8](https://datatracker.ietf.org/doc/html/rfc7519#section-8). По умолчанию `"HS256"`.
- `public_key` (`Optional[Union[str, pydantic.SecretStr]]`): Если алгоритм шифрования JWT требует пару ключей вместо простого секрета, здесь можно предоставить ключ для **дешифровки** JWT. Параметр `secret` всегда будет использоваться для **шифрования** JWT.

!!! tip "Почему это внутри функции?"
    Чтобы стратегии можно было создавать динамически с другими зависимостями, они должны предоставляться как вызываемые функции для аутентификационного бэкенда.

    Для `JWTStrategy`, поскольку она не требует зависимостей, функция может быть настолько простой, как указано выше.

## Пример с RS256

```py
from fastapi_users.authentication import JWTStrategy

PUBLIC_KEY = """-----BEGIN PUBLIC KEY-----
# Ваш открытый ключ RSA в формате PEM здесь
-----END PUBLIC KEY-----"""

PRIVATE_KEY = """-----BEGIN RSA PRIVATE KEY-----
# Ваш закрытый ключ RSA в формате PEM здесь
-----END RSA PRIVATE KEY-----"""

def get_jwt_strategy() -> JWTStrategy:
    return JWTStrategy(
        secret=PRIVATE_KEY, 
        lifetime_seconds=3600,
        algorithm="RS256",
        public_key=PUBLIC_KEY,
    )
```

## Выход из системы

При выходе из системы эта стратегия **ничего не сделает**. Действительно, JWT не может быть аннулирован на стороне сервера: он действителен до истечения срока его действия.