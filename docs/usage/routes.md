# Маршруты

Здесь вы найдете маршруты, предоставленные **FastAPI Users**. Обратите внимание, что вы также можете просмотреть их через [интерактивные документы API](https://fastapi.tiangolo.com/tutorial/first-steps/#interactive-api-docs).

## Роутер аутентификации

Каждый [метод аутентификации](../configuration/authentication/index.md), который вы [генерируете роутер для](../configuration/routers/auth.md), создаст следующие маршруты. Обратите внимание на префикс, который вы ему дали, особенно, если у вас есть несколько методов аутентификации.

### `POST /login`

Вход пользователя с использованием метода с именем `name`. Проверьте соответствующий [метод аутентификации](../configuration/authentication/index.md), чтобы увидеть успешный ответ.

!!! abstract "Данные (`application/x-www-form-urlencoded`)"
    ```
    username=king.arthur@camelot.bt&password=guinevere
    ```

!!! fail "`422 Ошибка валидации`"

!!! fail "`400 Плохой запрос`"
    Неверные учетные данные или пользователь неактивен.

    ```json
    {
        "detail": "LOGIN_BAD_CREDENTIALS"
    }
    ```

!!! fail "`400 Плохой запрос`"
    Пользователь не подтвержден.

    ```json
    {
        "detail": "LOGIN_USER_NOT_VERIFIED"
    }
    ```

### `POST /logout`

Выход аутентифицированного пользователя с использованием метода с именем `name`. Проверьте соответствующий [метод аутентификации](../configuration/authentication/index.md), чтобы увидеть успешный ответ.

!!! fail "`401 Unauthorized`"
    Отсутствует токен или пользователь неактивен.

!!! success "`200 OK`"
    Процесс выхода прошел успешно.

## Роутер регистрации

### `POST /register`

Регистрация нового пользователя. Вызовет [обработчик](../configuration/user-manager.md#on_after_register) `on_after_register` при успешной регистрации.

!!! abstract "Данные"
    ```json
    {
        "email": "king.arthur@camelot.bt",
        "password": "guinevere"
    }
    ```

!!! success "`201 Created`"
    ```json
    {
        "id": "57cbb51a-ab71-4009-8802-3f54b4f2e23",
        "email": "king.arthur@camelot.bt",
        "is_active": true,
        "is_superuser": false
    }
    ```

!!! fail "`422 Ошибка валидации`"

!!! fail "`400 Плохой запрос`"
    Пользователь с этим адресом электронной почты уже существует.

    ```json
    {
        "detail": "REGISTER_USER_ALREADY_EXISTS"
    }
    ```

!!! fail "`400 Плохой запрос`"
    Не удалось выполнить [проверку пароля](../configuration/user-manager.md#validate_password).

    ```json
    {
        "detail": {
            "code": "REGISTER_INVALID_PASSWORD",
            "reason": "Пароль должен содержать не менее 3 символов"
        }
    }
    ```

## Роутер сброса пароля

### `POST /forgot-password`

Запрос процедуры сброса пароля. Сгенерирует временный токен и вызовет [обработчик](../configuration/user-manager.md#on_after_forgot_password) `on_after_forgot_password`, если пользователь существует.

Чтобы предотвратить злоупотребление со стороны злоумышленников, которые могли бы угадать существующих пользователей в вашей базе данных, маршрут всегда вернет ответ `202 Accepted`, даже если запрашиваемого пользователя не существует.

!!! abstract "Данные"
    ```json
    {
        "email": "king.arthur@camelot.bt"
    }
    ```

!!! success "`202 Accepted`"

### `POST /reset-password`

Сброс пароля. Требуется токен, сгенерированный маршрутом `/forgot-password`.

!!! abstract "Данные"
    ```json
    {
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiOTIyMWZmYzktNjQwZi00MzcyLTg2ZDMtY2U2NDJjYmE1NjAzIiwiYXVkIjoiZmFzdGFwaS11c2VyczphdXRoIiwiZXhwIjoxNTcxNTA0MTkzfQ.M10bjOe45I5Ncu_uXvOmVV8QxnL-nZfcH96U90JaocI",
        "password": "merlin"
    }
    ```

!!! success "`200 OK`"

!!! fail "`422 Ошибка валидации`"

!!! fail "`400 Плохой запрос`"
    Неверный или устаревший токен.

    ```json
    {
        "detail": "RESET_PASSWORD_BAD_TOKEN"
    }
    ```

!!! fail "`400 Плохой запрос`"
    Не удалось выполнить [проверку пароля](../configuration/user-manager.md#validate_password).

    ```json
    {
        "detail": {
            "code": "RESET_PASSWORD_INVALID_PASSWORD",
            "reason": "Пароль должен содержать не менее 3 символов"
        }
    }
    ```

## Роутер верификации

### `POST /request-verify-token`

Запрос пользователю подтвердить свою электронную почту. Сгенерирует временный токен и вызовет [обработчик](../configuration/user-manager.md#on_after_request_verify) `on_after_request_verify`, если пользователь **существует**, **активен** и **еще не подтвержден**.

Чтобы предотвратить злоупотр

ебление со стороны злоумышленников, которые могли бы угадать существующих пользователей в вашей базе данных, маршрут всегда вернет ответ `202 Accepted`, даже если запрашиваемого пользователя не существует, не активен или уже подтвержден.

!!! abstract "Данные"
    ```json
    {
        "email": "king.arthur@camelot.bt"
    }
    ```

!!! success "`202 Accepted`"

### `POST /verify`

Подтверждение пользователя. Требуется токен, сгенерированный маршрутом `/request-verify-token`. Вызовет [обработчик](../configuration/user-manager.md#on_after_verify) `on_after_verify` при успешном выполнении.

!!! abstract "Данные"
    ```json
    {
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiOTIyMWZmYzktNjQwZi00MzcyLTg2ZDMtY2U2NDJjYmE1NjAzIiwiYXVkIjoiZmFzdGFwaS11c2VyczphdXRoIiwiZXhwIjoxNTcxNTA0MTkzfQ.M10bjOe45I5Ncu_uXvOmVV8QxnL-nZfcH96U90JaocI"
    }
    ```

!!! success "`200 OK`"

!!! fail "`422 Ошибка валидации`"

!!! fail "`400 Плохой запрос`"
    Неверный токен, несуществующий пользователь или не тот адрес электронной почты, который в данный момент установлен для пользователя.

    ```json
    {
        "detail": "VERIFY_USER_BAD_TOKEN"
    }
    ```

!!! fail "`400 Плохой запрос`"
    Пользователь уже подтвержден.

    ```json
    {
        "detail": "VERIFY_USER_ALREADY_VERIFIED"
    }
    ```
    
## Роутер OAuth

Каждый определенный вами роутер OAuth будет предоставлять два следующих маршрута.

### `GET /authorize`

Возвращает URL-адрес авторизации для службы OAuth, на которую вы должны перенаправить своего пользователя.

!!! abstract "Параметры запроса"
    * `scopes`: Необязательный список областей, о которых нужно спросить. Ожидаемый формат: `scopes=a&scopes=b`.

!!! success "`200 OK`"
    ```json
    {
        "authorization_url": "https://www.tintagel.bt/oauth/authorize?client_id=CLIENT_ID&scopes=a+b&redirect_uri=https://www.camelot.bt/oauth/callback"
    }
    ```

### `GET /callback`

Обрабатывает обратный вызов OAuth.

!!! abstract "Параметры запроса"
    * `code`: Код обратного вызова OAuth.
    * `state`: Токен состояния.
    * `error`: Ошибка OAuth.

В зависимости от ситуации может произойти несколько вещей:

* Учетная запись OAuth существует в базе данных и связана с пользователем:
    * Учетная запись OAuth обновляется в базе данных новым токеном доступа.
    * Пользователь аутентифицирован в соответствии с выбранным [методом аутентификации](../configuration/authentication/index.md).
* Учетная запись OAuth не существует в базе данных, но существует пользователь с тем же адресом электронной почты:
    * По умолчанию вызывается ошибка HTTP 400.
    * Если [флаг `associate_by_email` установлен в `True`](../configuration/oauth.md#existing-account-association) при объявлении роутера, учетная запись OAuth связывается с пользователем. Пользователь аутентифицирован в соответствии с выбранным [методом аутентификации](../configuration/authentication/index.md).
* Учетная запись OAuth не существует в базе данных, и пользователь с адресом электронной почты не существует:
    * Создается новый пользователь и связывается с учетной записью OAuth.
    * Пользователь аутентифицирован в соответствии с выбранным [методом аутентификации](../configuration/authentication/index.md).

!!! fail "`400 Bad Request`"
    Недопустимый токен.

!!! fail "`400 Bad Request`"
    Поставщик OAuth не вернул адрес электронной почты. Убедитесь, что этот поставщик возвращает адрес электронной почты через свой API, и вы запросили необходимую область.

    ```json
    {
        "detail": "OAUTH_NOT_AVAILABLE_EMAIL"
    }
    ```

!!! fail "`400 Bad Request`"
    Другой пользователь с тем же адресом электронной почты уже существует.

    ```json
    {
        "detail": "OAUTH_USER_ALREADY_EXISTS"
    }
    ```

!!! fail "`400 Bad Request`"
    Пользователь неактивен.

    ```json
    {
        "detail": "LOGIN_BAD_CREDENTIALS"
    }
    ```

## Роутер ассоциации OAuth

Каждый определенный вами роутер ассоциации OAuth будет предоставлять два следующих маршрута.

### `GET /authorize`

Возвращает URL-адрес авторизации для службы OAuth, на которую вы должны перенаправить своего пользователя.

!!! abstract "Параметры запроса"
    * `scopes`: Необязательный список областей, о которых нужно спросить. Ожидаемый формат: `scopes=a&scopes=b`.

!!! fail "`401 Unauthorized`"
    Отсутствует токен или неактивный пользователь.

!!! success "`200 OK`"
    ```json
    {
        "authorization_url": "https://www.tintagel.bt/oauth/authorize?client_id=CLIENT_ID&scopes=a+b&redirect_uri=https://www.camelot.bt/oauth/callback"
    }
    ```

### `GET /callback`

Обрабатывает обратный вызов OAuth и добавляет учетную запись OAuth к текущему аутентифицированному активному пользователю.

!!! abstract "Параметры запроса"
    * `code`: Код обратного вызова OAuth.
    * `state`: Токен состояния.
    * `error`: Ошибка OAuth.

!!! fail "`401 Unauthorized`"
    Отсутствует токен или неактивный пользователь.

!!! fail "`400 Bad Request`"
    Недопустимый токен.

!!! fail "`400 Bad Request`"
    Поставщик OAuth не вернул адрес электронной почты. Убедитесь, что этот поставщик возвращает адрес электронной почты через свой API, и вы запросили необходимую область.

    ```json
    {
        "detail": "OAUTH_NOT_AVAILABLE_EMAIL"
    }
    ```

!!! success "`200 OK`"
    ```json
    {
        "id": "57cbb51a-ab71-4009-8802-3f54b4f2e23",
        "email": "king.arthur@tintagel.bt",
        "is_active": true,
        "is_superuser": false,
        "oauth_accounts": [
            {
                "id": "6c98caf5-9bc5-4c4f-8a45-a0ae0c40cd77",
                "oauth_name": "TINTAGEL",
                "access_token": "ACCESS_TOKEN",
                "expires_at": "1641040620",
                "account_id": "king_arthur_tintagel",
                "account_email": "king.arthur@tintagel.bt"
            }
        ]
    }
    ```

## Роутер пользователей

### `GET /me`

Возвращает текущего аутентифицированного активного пользователя.

!!! success "`200 OK`"
    ```json
    {
        "id": "57cbb51a-ab71-4009-8802-3f54b4f2

e23",
        "email": "king.arthur@camelot.bt",
        "is_active": true,
        "is_superuser": false
    }
    ```

!!! fail "`401 Unauthorized`"
    Отсутствует токен или неактивный пользователь.

### `PATCH /me`

Обновляет текущего аутентифицированного активного пользователя.

!!! abstract "Данные"
    ```json
    {
        "email": "king.arthur@tintagel.bt",
        "password": "merlin"
    }
    ```

!!! success "`200 OK`"
    ```json
    {
        "id": "57cbb51a-ab71-4009-8802-3f54b4f2e23",
        "email": "king.arthur@tintagel.bt",
        "is_active": true,
        "is_superuser": false
    }
    ```

!!! fail "`401 Unauthorized`"
    Отсутствует токен или неактивный пользователь.

!!! fail "`400 Bad Request`"
    [Проверка пароля](../configuration/user-manager.md#validate_password) не удалась.

    ```json
    {
        "detail": {
            "code": "UPDATE_USER_INVALID_PASSWORD",
            "reason": "Пароль должен быть не менее 3 символов"
        }
    }
    ```

!!! fail "`400 Bad Request`"
    Пользователь с этим адресом электронной почты уже существует.

    ```json
    {
        "detail": "UPDATE_USER_EMAIL_ALREADY_EXISTS"
    }
    ```

!!! fail "`422 Ошибка валидации`"

### `GET /{user_id}`

Возвращает пользователя с идентификатором `user_id`.

!!! success "`200 OK`"
    ```json
    {
        "id": "57cbb51a-ab71-4009-8802-3f54b4f2e23",
        "email": "king.arthur@camelot.bt",
        "is_active": true,
        "is_superuser": false
    }
    ```

!!! fail "`401 Unauthorized`"
    Отсутствует токен или неактивный пользователь.

!!! fail "`403 Forbidden`"
    Не является суперпользователем.

!!! fail "`404 Not found`"
    Пользователь не существует.

### `PATCH /{user_id}`

Обновляет пользователя с идентификатором `user_id`.

!!! abstract "Данные"
    ```json
    {
        "email": "king.arthur@tintagel.bt",
        "password": "merlin",
        "is_active": false,
        "is_superuser": true
    }
    ```

!!! success "`200 OK`"
    ```json
    {
        "id": "57cbb51a-ab71-4009-8802-3f54b4f2e23",
        "email": "king.arthur@camelot.bt",
        "is_active": false,
        "is_superuser": true
    }
    ```

!!! fail "`401 Unauthorized`"
    Отсутствует токен или неактивный пользователь.

!!! fail "`403 Forbidden`"
    Не является суперпользователем.

!!! fail "`404 Not found`"
    Пользователь не существует.

!!! fail "`400 Bad Request`"
    [Проверка пароля](../configuration/user-manager.md#validate_password) не удалась.

    ```json
    {
        "detail": {
            "code": "UPDATE_USER_INVALID_PASSWORD",
            "reason": "Пароль должен быть не менее 3 символов"
        }
    }
    ```

!!! fail "`400 Bad Request`"
    Пользователь с этим адресом электронной почты уже существует.
    ```json
    {
        "detail": "UPDATE_USER_EMAIL_ALREADY_EXISTS"
    }
    ```

### `DELETE /{user_id}`

Удаляет пользователя с идентификатором `user_id`.

!!! success "`204 No content`"

!!! fail "`401 Unauthorized`"
    Отсутствует токен или неактивный пользователь.

!!! fail "`403 Forbidden`"
    Не является суперпользователем.

!!! fail "`404 Not found`"
    Пользователь не существует.