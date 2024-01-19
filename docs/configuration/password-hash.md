# Хеширование пароля

По умолчанию FastAPI Users будет использовать [алгоритм BCrypt](https://en.wikipedia.org/wiki/Bcrypt) для **хеширования и соления** паролей перед сохранением их в базе данных.

Реализация предоставляется [Passlib](https://passlib.readthedocs.io/en/stable/index.html), проверенной временем библиотекой на языке Python для хеширования паролей.

## Настройка `CryptContext`

Если вам нужна поддержка других алгоритмов хеширования, вы можете настроить [`CryptContext` объект из Passlib](https://passlib.readthedocs.io/en/stable/lib/passlib.context.html#the-cryptcontext-class).

Для этого вам нужно создать экземпляр класса `PasswordHelper` и передать ему ваш `CryptContext`. В приведенном ниже примере показано, как вы можете создать `CryptContext` для поддержки алгоритма Argon2 при устаревании BCrypt.

```py
from fastapi_users.password import PasswordHelper
from passlib.context import CryptContext

context = CryptContext(schemes=["argon2", "bcrypt"], deprecated="auto")
password_helper = PasswordHelper(context)
```

Наконец, передайте переменную `password_helper` при создании вашего `UserManager`:

```py
async def get_user_manager(user_db=Depends(get_user_db)):
    yield UserManager(user_db, password_helper)
```

!!! info "Хеши паролей автоматически обновляются"
    FastAPI Users заботится о том, чтобы обновлять хеш пароля до более нового алгоритма при необходимости.

    Обычно, когда пользователь входит в систему, мы проверяем, устарел ли алгоритм хеша пароля.

    Если это так, мы воспользуемся возможностью иметь пароль в виде обычного текста (поскольку пользователь только что вошел в систему!), чтобы хешировать его с использованием более надежного алгоритма и обновить его в базе данных.

!!! warning "Зависимости для альтернативных алгоритмов не устанавливаются по умолчанию"
    FastAPI Users не установит необходимые зависимости, чтобы сделать другие алгоритмы, такие как Argon2, работающими. Вам нужно установить их самостоятельно.

## Полная настройка

Если вы не хотите использовать Passlib вообще – **что мы не рекомендуем, если вы не уверены в том, что делаете** — вы можете реализовать свой собственный класс `PasswordHelper`, при условии, что он реализует `PasswordHelperProtocol` и его методы.

```py
from typing import Tuple

from fastapi_users.password import PasswordHelperProtocol

class PasswordHelper(PasswordHelperProtocol):
    def verify_and_update(
        self, plain_password: str, hashed_password: str
    ) -> Tuple[bool, str]:
        ...

    def hash(self, password: str) -> str:
        ...

    def generate(self) -> str:
        ...
```