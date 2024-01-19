# Полный пример

Вот полный рабочий пример с аутентификацией через JWT, чтобы помочь вам начать.

!!! warning
    Обратите внимание, что **SECRET** должен быть изменен на надежную парольную фразу.
    Небезопасные пароли могут предоставить злоумышленникам полный доступ к вашей базе данных.

## SQLAlchemy

[Открыть :material-open-in-new:](https://github.com/fastapi-users/fastapi-users/tree/master/examples/sqlalchemy)

=== "requirements.txt"

    ```
    --8<-- "examples/sqlalchemy/requirements.txt"
    ```

=== "main.py"

    ```py
    --8<-- "examples/sqlalchemy/main.py"
    ```

=== "app/app.py"

    ```py
    --8<-- "examples/sqlalchemy/app/app.py"
    ```

=== "app/db.py"

    ```py
    --8<-- "examples/sqlalchemy/app/db.py"
    ```

=== "app/schemas.py"

    ```py
    --8<-- "examples/sqlalchemy/app/schemas.py"
    ```

=== "app/users.py"

    ```py
    --8<-- "examples/sqlalchemy/app/users.py"
    ```

## Beanie

[Открыть :material-open-in-new:](https://github.com/fastapi-users/fastapi-users/tree/master/examples/beanie)

=== "requirements.txt"

    ```
    --8<-- "examples/beanie/requirements.txt"
    ```

=== "main.py"

    ```py
    --8<-- "examples/beanie/main.py"
    ```

=== "app/app.py"

    ```py
    --8<-- "examples/beanie/app/app.py"
    ```

=== "app/db.py"

    ```py
    --8<-- "examples/beanie/app/db.py"
    ```

=== "app/schemas.py"

    ```py
    --8<-- "examples/beanie/app/schemas.py"
    ```

=== "app/users.py"

    ```py
    --8<-- "examples/beanie/app/users.py"
    ```

## Что дальше?

Вы готовы начать! Обязательно ознакомьтесь с разделом [Использование](../usage/routes.md), чтобы понять, как работать с **FastAPI Users**.