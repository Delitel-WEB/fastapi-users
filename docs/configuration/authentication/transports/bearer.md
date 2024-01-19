# Bearer

С этим транспортом ожидается, что токен будет внутри заголовка HTTP-запроса `Authorization` с схемой `Bearer`. Он особенно подходит для чистого взаимодействия с API или мобильных приложений.

## Конфигурация

```py
from fastapi_users.authentication import BearerTransport

bearer_transport = BearerTransport(tokenUrl="auth/jwt/login")
```

Как видите, создание экземпляра довольно просто. Он принимает следующие аргументы:

* `tokenUrl` (`str`): Точный путь вашего конечного точки входа. Это позволит интерактивной документации автоматически обнаруживать его и получать работающую кнопку *Authorize*. В большинстве случаев вам, вероятно, потребуется **относительный** путь, а не абсолютный. Подробности можно прочитать в [документации FastAPI](https://fastapi.tiangolo.com/tutorial/security/first-steps/#fastapis-oauth2passwordbearer).

## Вход в систему

Этот метод вернет следующую форму при успешном входе:

!!! success "`200 OK`"
    ```json
    {
        "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiOTIyMWZmYzktNjQwZi00MzcyLTg2ZDMtY2U2NDJjYmE1NjAzIiwiYXVkIjoiZmFzdGFwaS11c2VyczphdXRoIiwiZXhwIjoxNTcxNTA0MTkzfQ.M10bjOe45I5Ncu_uXvOmVV8QxnL-nZfcH96U90JaocI",
        "token_type": "bearer"
    }
    ```

> Проверьте документацию о [маршруте входа в систему](../../../usage/routes.md#post-login).

## Выход из системы

!!! success "`204 No content`"

## Аутентификация

Этот метод ожидает, что вы предоставите аутентификацию `Bearer` с действительным токеном, соответствующим вашей стратегии.

```bash
curl http://localhost:9000/protected-route -H'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiOTIyMWZmYzktNjQwZi00MzcyLTg2ZDMtY2U2NDJjYmE1NjAzIiwiYXVkIjoiZmFzdGFwaS11c2VyczphdXRoIiwiZXhwIjoxNTcxNTA0MTkzfQ.M10bjOe45I5Ncu_uXvOmVV8QxnL-nZfcH96U90JaocI'
```