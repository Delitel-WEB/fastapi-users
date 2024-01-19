# Менеджер пользователей

Класс `UserManager` представляет собой основную логику FastAPI Users. Мы предоставляем класс `BaseUserManager`, который вам следует расширить для установки некоторых параметров и определения логики, например, когда пользователь только что зарегистрировался или забыл свой пароль.

Он разработан с учетом легкости расширения и настройки, чтобы вы могли интегрировать свою собственную логику.

## Создайте свой класс `UserManager`

Вам следует определить свою собственную версию класса `UserManager`, чтобы установить различные параметры.

```py hl_lines="12-27"
--8<-- "docs/src/user_manager.py"
```

Как видите, здесь нужно определить различные атрибуты и методы. Полный список из них приведен ниже.

!!! note "Типизация: Ожидаются обобщенные типы User и ID"
    Вы видите, что мы определяем два обобщенных типа при расширении базового класса:

    * `User`, который является моделью пользователя, которую мы определили в части базы данных.
    * ID, который должен соответствовать типу ID, используемому в вашей модели. Здесь мы выбрали UUID, но это может быть что угодно, например, целое число или MongoDB ObjectID.

    Это поможет вам получить **хорошую проверку типов и автозаполнение** при реализации собственных методов.

### Смешивание парсера ID

Поскольку ID пользователя является полностью обобщенным, нам нужен способ **надежно его разбирать, когда он будет поступать из запросов API**, обычно в виде атрибутов пути URL.

Вот почему мы добавили `UUIDIDMixin` в приведенном выше примере. Он реализует метод `parse_id`, обеспечивая, что UUID действительны и правильно разбираются.

Конечно, важно, чтобы эта логика **соответствовала типу вашего ID**. Чтобы помочь вам с этим, мы предоставляем смешивания для наиболее распространенных случаев:

* `UUIDIDMixin` для UUID ID.
* `IntegerIDMixin` для целочисленного ID.
* `ObjectIDIDMixin` (предоставляется `fastapi_users_db_beanie`) для MongoDB ObjectID.

!!! tip "Порядок наследования имеет значение"
    Обратите внимание в вашем примере, что **смешивание идет первым в нашем наследовании `UserManager`**. Из-за порядка разрешения методов (MRO) Python, самый левый элемент имеет приоритет.

Если вам нужен другой тип ID, вы можете просто перегрузить метод `parse_id` в своем классе `UserManager`:

```py
from fastapi_users import BaseUserManager, InvalidID


class UserManager(BaseUserManager[User, MyCustomID]):
    def parse_id(self, value: Any) -> MyCustomID:
        try:
            return MyCustomID(value)
        except ValueError as e:
            raise InvalidID() from e  # (1)!
```

1. Если ID невозможно разобрать в желаемый тип, вам нужно вызвать исключение `InvalidID`.

## Создание зависимости `get_user_manager`

Класс `UserManager` будет внедрен во время выполнения с использованием зависимости FastAPI. Таким образом, вы можете выполнять его в сеансе базы данных или заменять его на заглушку во время тестирования.

```py hl_lines="30-31"
--8<-- "docs/src/user_manager.py"
```

Обратите внимание, что мы используем зависимость `get_user_db`, которую мы определили ранее, для внедрения экземпляра базы данных.

## Настройка атрибутов и методов

### Атрибуты

* `reset_password_token_secret`: Секрет для кодирования токена сброса пароля. **Используйте надежную фразу-пароль и держите ее в безопасности.**
* `reset_password_token_lifetime_seconds`: Срок действия токена сброса пароля. По умолчанию 3600.
* `reset_password_token_audience`: Аудитория JWT токена сброса пароля. По умолчанию `fastapi-users:reset`.
* `verification_token_secret`: Секрет для кодирования токена подтверждения. **Используйте надежную фразу-пароль и держите ее в безопасности.**
* `verification_token_lifetime_seconds`: Срок действия токена подтверждения. По умолчанию 3600.
* `verification_token_audience`: Аудитория JWT токена подтверждения. По умолчанию `fastapi-users:verify`.

### Методы

#### `validate_password`

Проверка пароля.

**Аргументы**

* `password` (`str`): проверяемый пароль.
* `user` (`Union[UserCreate, User]`): модель пользователя, для которой в настоящее время проверяется пароль. Полезно, если вы хотите проверить, что пароль не содержит имя или дату рождения пользователя, например.

**

Вывод**

Эта функция должна вернуть `None`, если пароль допустим, или вызвать `InvalidPasswordException`, если нет. Это исключение ожидает аргумент `reason`, указывающий причину недопустимости пароля. Он будет частью ответа с ошибкой.

**Пример**

```py
from fastapi_users import BaseUserManager, InvalidPasswordException, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def validate_password(
        self,
        password: str,
        user: Union[UserCreate, User],
    ) -> None:
        if len(password) < 8:
            raise InvalidPasswordException(
                reason="Password should be at least 8 characters"
            )
        if user.email in password:
            raise InvalidPasswordException(
                reason="Password should not contain e-mail"
            )
```

#### `on_after_register`

Выполнение логики после успешной регистрации пользователя.

Обычно вам захочется **отправить приветственное письмо** или добавить его в вашу аналитику маркетинга.

**Аргументы**

* `user` (`User`): зарегистрированный пользователь.
* `request` (`Optional[Request]`): необязательный объект запроса FastAPI, который вызвал операцию. По умолчанию None.

**Пример**

```py
from fastapi_users import BaseUserManager, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def on_after_register(self, user: User, request: Optional[Request] = None):
        print(f"User {user.id} has registered.")
```

#### `on_after_update`

Выполнение логики после успешного обновления пользователя.

Это может быть полезно, например, если вы хотите обновить своего пользователя в аналитике данных или платформе успеха клиента.

**Аргументы**

* `user` (`User`): обновленный пользователь.
* `update_dict` (`Dict[str, Any]`): словарь с обновленными полями пользователя.
* `request` (`Optional[Request]`): необязательный объект запроса FastAPI, который вызвал операцию. По умолчанию None.

**Пример**

```py
from fastapi_users import BaseUserManager, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def on_after_update(
        self,
        user: User,
        update_dict: Dict[str, Any],
        request: Optional[Request] = None,
    ):
        print(f"User {user.id} has been updated with {update_dict}.")
```

#### `on_after_login`

Выполнение логики после успешного входа пользователя.

Это может быть полезно для пользовательской логики или процессов, запускаемых новыми входами, например, ежедневная награда за вход или для аналитики.

**Аргументы**

* `user` (`User`): обновленный пользователь.
* `request` (`Optional[Request]`): необязательный объект запроса FastAPI, который вызвал операцию. По умолчанию None.
* `response` (`Optional[Response]`): Необязательный ответ, созданный транспортом. По умолчанию None.

**Пример**

```py
from fastapi_users import BaseUserManager, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def on_after_login(
        self,
        user: User,
        request: Optional[Request] = None,
        response: Optional[Response] = None,
    ):
        print(f"User {user.id} logged in.")
```

#### `on_after_request_verify`

Выполнение логики после успешного запроса на подтверждение.

Обычно вам захочется **отправить электронное письмо** со ссылкой (и токеном), который позволит пользователю подтвердить свой адрес электронной почты.

**Аргументы**

* `user` (`User`): пользователь для подтверждения.
* `token` (`str`): токен подтверждения.
* `request` (`Optional[Request]`): необязательный объект запроса FastAPI, который вызвал операцию. По умолчанию None.

**Пример**

```py
from fastapi_users import BaseUserManager, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def on_after_request_verify(
        self, user: User, token: str, request: Optional[Request] = None
    ):
        print(f"Запрос на подтверждение для пользователя {user.id}. Токен подтверждения: {token}")
```

#### `on_after_verify`

Выполнение логики после успешного подтверждения пользователя.

Это может быть полезно, если вы хотите отправить еще одно электронное письмо или сохранить эту информацию в аналитике данных или платформе успеха клиента.

**Аргументы**

* `user` (`User`): подтвержденный пользователь.
* `request` (`Optional[Request]`): необязательный объект запроса FastAPI, который вызвал операцию. По умолчанию None.

**Пример**

```py
from fastapi_users import BaseUserManager, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def on_after_verify(
        self, user: User, request: Optional[Request] = None
    ):
        print(f"Пользователь {user.id} был подтвержден")
```

#### `on_after_forgot_password`

Выполнение логики после успешного запроса сброса пароля.

Обычно вам захочется **отправить электронное письмо** со ссылкой (и токеном), который позволяет пользователю сбросить свой пароль.

**Аргументы**

* `user` (`User`): пользователь, который забыл свой пароль.
* `token` (`str`): токен сброса пароля.
* `request` (`Optional[Request]`): необязательный объект запроса FastAPI, который вызвал операцию. По умолчанию None.

**Пример**

```py
from fastapi_users import BaseUserManager, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def on_after_forgot_password(
        self, user: User, token: str, request: Optional[Request] = None
    ):
        print(f"Пользователь {user.id} забыл свой пароль. Токен сброса: {token}")
```

#### `on_after_reset_password`

Выполнение логики после успешного сброса пароля.

Например, вы можете **отправить электронное письмо** соответствующему пользователю, предупредив, что его пароль был изменен, и что он должен предпринять действия, если считает, что его аккаунт взломан.

**Аргументы**

* `user` (`User`): пользователь, который сбросил свой пароль.
* `request` (`Optional[Request]`): необязательный объект запроса FastAPI, который вызвал операцию. По умолчанию None.

**Пример**

```py
from fastapi_users import BaseUserManager, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def on_after_reset_password(self, user: User, request: Optional[Request] = None):
        print(f"Пользователь {user.id} сбросил свой пароль.")
```

#### `on_before_delete`

Выполнение логики перед удалением пользователя.

Например, вам может потребоваться **проверить целостность ресурсов пользователя**, чтобы увидеть, необходимо ли помечать как неактивные связанные с пользователем ресурсы или удалять их рекурсивно.

**Аргументы**

* `user` (`User`): пользователь, который будет удален.
* `request` (`Optional[Request]`): необязательный объект запроса FastAPI, который вызвал операцию. По умолчанию None.

**Пример**

```py
from fastapi_users import BaseUserManager, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def on_before_delete(self, user: User, request: Optional[Request] = None):
        print(f"Пользователь {user.id} собирается быть удален")
```

#### `on_after_delete`

Выполнение логики после удаления пользователя.

Например, вы можете захотеть **отправить электронное письмо** администратору о событии.

**Аргументы**

* `user` (`User`): пользователь, который будет удален.
* `request` (`Optional[Request]`): необязательный объект запроса FastAPI, который вызвал операцию. По умолчанию None.

**Пример**

```py
from fastapi_users import BaseUserManager, UUIDIDMixin


class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    # ...
    async def on_after_delete(self, user: User, request: Optional[Request] = None):
        print(f"Пользователь {user.id} успешно удален")
```