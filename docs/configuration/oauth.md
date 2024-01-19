# OAuth2

FastAPI Users предоставляет дополнительную поддержку аутентификации OAuth2. Он зависит от [библиотеки OAuth HTTPX](https://frankie567.github.io/httpx-oauth/), которая представляет собой чистую асинхронную реализацию OAuth2.

## Установка

Вы должны установить библиотеку с дополнительными зависимостями для OAuth:

```sh
pip install 'fastapi-users[sqlalchemy,oauth]'
```

или

```sh
pip install 'fastapi-users[beanie,oauth]'
```

## Настройка

### Создание экземпляра клиента OAuth2

Сначала вам нужно получить экземпляр клиента HTTPX OAuth. [Прочтите документацию](https://frankie567.github.io/httpx-oauth/oauth2/) для получения дополнительной информации.

```py
from httpx_oauth.clients.google import GoogleOAuth2

google_oauth_client = GoogleOAuth2("CLIENT_ID", "CLIENT_SECRET")
```

### Настройка адаптера базы данных

#### SQLAlchemy

Вам нужно определить модель SQLAlchemy для хранения учетных записей OAuth. Мы предоставляем базовую модель для этого:

```py hl_lines="5 19-20 24-26 43-44"
--8<-- "docs/src/db_sqlalchemy_oauth.py"
```

Обратите внимание, что мы также вручную добавили `relationship` на `User`, чтобы SQLAlchemy мог правильно извлекать учетные записи OAuth пользователя.

Кроме того, при создании адаптера базы данных нам нужно передать эту модель SQLAlchemy в качестве третьего аргумента.

!!! tip "Первичный ключ определен как UUID"
    По умолчанию мы используем UUID в качестве первичного ключа ID для вашего пользователя. Если вы хотите использовать другой тип, например, автоматически увеличиваемое целое число, вы можете использовать `SQLAlchemyBaseOAuthAccountTable` в качестве базового класса и определить свой собственный столбец `id` и `user_id`.

    ```py
    class OAuthAccount(SQLAlchemyBaseOAuthAccountTable[int], Base):
        id: Mapped[int] = mapped_column(Integer, primary_key=True)

        @declared_attr
        def user_id(cls) -> Mapped[int]:
            return mapped_column(Integer, ForeignKey("user.id", ondelete="cascade"), nullable=False)

    ```

    Обратите внимание, что `SQLAlchemyBaseOAuthAccountTable` ожидает обобщенный тип для определения фактического типа ID, который вы используете.

#### Beanie

Преимущество MongoDB заключается в том, что вы легко можете встраивать подобъекты в один документ. Поэтому конфигурация для Beanie довольно проста. Все, что нам нужно сделать, это определить еще один класс для структурирования объекта учетной записи OAuth.

```py hl_lines="5 15-16 20"
--8<-- "docs/src/db_beanie_oauth.py"
```

Стоит отметить, что `OAuthAccount` **не является документом Beanie**, а является моделью Pydantic, которую мы будем встраивать в документ `User` через массив `oauth_accounts`.

### Создание маршрутов

После создания экземпляра `FastAPIUsers` вы можете заставить его создать единственный маршрут OAuth для заданного клиента **и** бэкенда аутентификации.

```py
app.include_router(
    fastapi_users.get_oauth_router(google_oauth_client, auth_backend, "SECRET"),
    prefix="/auth/google",
    tags=["auth"],
)
```

!!! tip
    Если у вас есть несколько клиентов OAuth и/или несколько бэкендов аутентификации, вам нужно создать маршрут для каждой пары, которую вы хотите поддерживать.

#### Ассоциация существующей учетной записи

Если учетная запись с таким же адресом электронной почты уже существует, по умолчанию будет вызвана ошибка HTTP 400.

Однако вы можете выбрать автоматическую привязку этой учетной записи OAuth к существующей учетной записи пользователя, установив флаг `associate_by_email`:

```py
app.include_router(
    fastapi_users.get_oauth_router(
        google_oauth_client,
        auth_backend,
        "SECRET",
        associate_by_email=True,
    ),
    prefix="/auth/google",
    tags=["auth"],
)
```

Тем не менее, следует помнить, что это может привести к нарушениям безопасности, если провайдер OAuth не проверяет адреса электронной почты. Как?

* Допустим, ваше приложение поддерживает OAuth-провайдер *Merlinbook*, который не проверяет адреса электронной почты.
* Представьте себе, что пользователь регистрируется в вашем приложении с адресом электронной почты `lancelot@camelot.bt`.
* Теперь злонамеренный пользователь создает учетную запись в *Merlinbook* с тем же адресом электронной почты. Без проверки электронной почты злонамеренный пользователь может использовать эту учетную запись без ограничений.
* Злонамеренный пользователь проходит аутентификацию с использованием OAuth *Merlinbook* в вашем приложении, которая автоматически ассоциируется с существующим `lancelot@camelot.bt`.
* Теперь злонамеренный пользователь имеет полный доступ к учетной записи пользователя в ваш

ем приложении 😞

#### Маршрут ассоциации для аутентифицированных пользователей

Мы также предоставляем маршрут для ассоциирования уже аутентифицированного пользователя с учетной записью OAuth. После этой ассоциации пользователь сможет аутентифицироваться с этим провайдером OAuth.

```py
app.include_router(
    fastapi_users.get_oauth_associate_router(google_oauth_client, UserRead, "SECRET"),
    prefix="/auth/associate/google",
    tags=["auth"],
)
```

Обратите внимание, что, как и для [маршрута пользователей](./routers/users.md), вы должны передать схему Pydantic `UserRead`.

#### Установка `is_verified` в `True` по умолчанию

!!! tip "Этот раздел полезен только при настройке проверки электронной почты"
    Вы можете прочитать больше об этой функции [здесь](./routers/verify.md).

Когда новый пользователь регистрируется с провайдером OAuth, флаг `is_verified` устанавливается в `False`, что требует от пользователя подтверждения своего адреса электронной почты.

Вы можете выбрать доверять адресу электронной почты, предоставленному провайдером OAuth, и установить флаг `is_verified` в `True` после регистрации. Вы можете сделать это, установив аргумент `is_verified_by_default`:

```py
app.include_router(
    fastapi_users.get_oauth_router(
        google_oauth_client,
        auth_backend,
        "SECRET",
        is_verified_by_default=True,
    ),
    prefix="/auth/google",
    tags=["auth"],
)
```

!!! danger "Убедитесь, что вы можете доверять провайдеру OAuth"
    Убедитесь, что используемый вами провайдер OAuth **проверяет** адрес электронной почты, прежде чем включать этот флаг.

### Полный пример

!!! warning
    Обратите внимание, что **SECRET** должен быть изменен на надежную пароль. Небезопасные пароли могут предоставить злоумышленникам полный доступ к вашей базе данных.

#### SQLAlchemy

[Открыть :material-open-in-new:](https://github.com/fastapi-users/fastapi-users/tree/master/examples/sqlalchemy-oauth)

=== "requirements.txt"

    ```
    --8<-- "examples/sqlalchemy-oauth/requirements.txt"
    ```

=== "main.py"

    ```py
    --8<-- "examples/sqlalchemy-oauth/main.py"
    ```

=== "app/app.py"

    ```py
    --8<-- "examples/sqlalchemy-oauth/app/app.py"
    ```

=== "app/db.py"

    ```py
    --8<-- "examples/sqlalchemy-oauth/app/db.py"
    ```

=== "app/schemas.py"

    ```py
    --8<-- "examples/sqlalchemy-oauth/app/schemas.py"
    ```

=== "app/users.py"

    ```py
    --8<-- "examples/sqlalchemy-oauth/app/users.py"
    ```

#### Beanie

[Открыть :material-open-in-new:](https://github.com/fastapi-users/fastapi-users/tree/master/examples/beanie-oauth)

=== "requirements.txt"

    ```
    --8<-- "examples/beanie-oauth/requirements.txt"
    ```

=== "main.py"

    ```py
    --8<-- "examples/beanie-oauth/main.py"
    ```

=== "app/app.py"

    ```py
    --8<-- "examples/beanie-oauth/app/app.py"
    ```

=== "app/db.py"

    ```py
    --8<-- "examples/beanie-oauth/app/db.py"
    ```

=== "app/schemas.py"

    ```py
    --8<-- "examples/beanie-oauth/app/schemas.py"
    ```

=== "app/users.py"

    ```py
    --8<-- "examples/beanie-oauth/app/users.py"
    ```