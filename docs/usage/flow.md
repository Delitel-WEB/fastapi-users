# Flow

На этой странице будет представлен полный поток регистрации и аутентификации после установки **FastAPI Users**. Каждый пример будет представлен с использованием `cURL` и примера с `axios`.

## 1. Регистрация

Первый шаг, конечно, это зарегистрироваться в качестве пользователя.

### Запрос

=== "cURL"
    ``` bash
    curl \
    -H "Content-Type: application/json" \
    -X POST \
    -d "{\"email\": \"king.arthur@camelot.bt\",\"password\": \"guinevere\"}" \
    http://localhost:8000/auth/register
    ```

=== "axios"
    ```ts
    axios.post('http://localhost:8000/auth/register', {
        email: 'king.arthur@camelot.bt',
        password: 'guinevere',
    })
    .then((response) => console.log(response))
    .catch((error) => console.log(error));
    ```

### Ответ

Вы получите JSON-ответ, который будет выглядеть так:

```json
{
    "id": "4fd3477b-eccf-4ee3-8f7d-68ad72261476",
    "email": "king.arthur@camelot.bt",
    "is_active": true,
    "is_superuser": false
}
```

!!! info
    Несколько важных моментов:

    * Если вы определили другие обязательные поля в своей модели `User` (например, имя или день рождения), вы должны предоставить их в теле запроса.
    * Пользователь активен по умолчанию.
    * Пользователь не может установить `is_active` или `is_superuser` самостоятельно при регистрации. Это может сделать только суперпользователь, обновив пользователя с помощью PATCH.

## 2. Логин

Теперь вы можете войти как этот новый пользователь.

Вы можете создать [маршрут входа](../configuration/routers/auth.md) для каждого [бэкенда аутентификации](../configuration/authentication/index.md). Каждый бэкенд будет иметь различный ответ.

### Bearer + JWT

#### Запрос

=== "cURL"
    ``` bash
    curl \
    -H "Content-Type: multipart/form-data" \
    -X POST \
    -F "username=king.arthur@camelot.bt" \
    -F "password=guinevere" \
    http://localhost:8000/auth/jwt/login
    ```

=== "axios"
    ```ts
    const formData = new FormData();
    formData.set('username', 'king.arthur@camelot.bt');
    formData.set('password', 'guinevere');
    axios.post(
        'http://localhost:8000/auth/jwt/login',
        formData,
        {
            headers: {
                'Content-Type': 'multipart/form-data',
            },
        },
    )
    .then((response) => console.log(response))
    .catch((error) => console.log(error));
    ```

!!! warning
    Обратите внимание, что здесь мы не отправляем его как JSON-полезную нагрузку, а вместо этого используем **форму данных**. Кроме того, электронная почта предоставляется полем с именем **`username`**.

#### Ответ

Вы получите JSON-ответ, который будет выглядеть так:

```json
{
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiNGZkMzQ3N2ItZWNjZi00ZWUzLThmN2QtNjhhZDcyMjYxNDc2IiwiYXVkIjoiZmFzdGFwaS11c2VyczphdXRoIiwiZXhwIjoxNTg3ODE4NDI5fQ.anO3JR8-WYCozZ4_2-PQ2Ov9O38RaLP2RAzQIiZhteM",
    "token_type": "bearer"
}
```

Вы можете использовать этот токен для выполнения аутентифицированных запросов от имени пользователя `king.arthur@camelot.bt`. Мы рассмотрим это в следующем разделе.

### Cookie + JWT

#### Запрос

=== "cURL"
    ``` bash
    curl \
    -v \
    -H "Content-Type: multipart/form-data" \
    -X POST \
    -F "username=king.arthur@camelot.bt" \
    -F "password=guinevere" \
    http://localhost:8000/auth/cookie/login
    ```

=== "axios"
    ```ts
    const formData = new FormData();
    formData.set('username', 'king.arthur@camelot.bt');
    formData.set('password', 'guinevere');
    axios.post(
        'http://localhost:8000/auth/cookie/login',
        formData,
        {
            headers: {
                'Content-Type': 'multipart/form-data',
            },
        },
    )
    .then((response) => console.log(response))
    .catch((error) => console.log(error));
    ```

!!! warning
    Обратите внимание, что здесь мы не отправляем его как JSON-полезную нагрузку, а вместо этого используем **форму данных**. Кроме того, электронная почта предоставляется полем с именем **`username`**.

#### Ответ

Вы получите пустой ответ. Однако ответ будет содержать заголовок `Set-Cookie` (поэтому мы добавили опцию `-v` в `cURL`, чтобы их увидеть).

```
set-cookie: fastapiusersauth=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiYzYwNjBmMTEtNTM0OS00YTI0LThiNGEtYTJhODc1ZGM1Mzk1IiwiYXVkIjoiZmFzdGFwaS11c2VyczphdXRoIiwiZXhwIjoxNTg3ODE4OTQ3fQ.qNA4oPVYhoqrJIk-z

vAyEfEVoEnP156G30H_SWEU0sU; HttpOnly; Max-Age=3600; Path=/; Secure
```

Вы можете делать аутентифицированные запросы от имени пользователя `king.arthur@camelot.bt`, установив заголовок `Cookie` с этим файлом cookie.

!!! tip
    Бэкенд cookie более подходит для браузеров, так как они обрабатывают их автоматически. Это означает, что если вы делаете запрос на вход в браузере, он автоматически сохранит файл cookie и автоматически отправит его в последующих запросах.

## 3. Получить мой профиль

Теперь, когда мы можем проходить аутентификацию, мы можем получить данные своего профиля. В зависимости от [бэкенда аутентификации](../configuration/authentication/index.md), метод аутентификации запроса будет различаться. Мы будем использовать JWT.

### Запрос

=== "cURL"
    ``` bash
    export TOKEN="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiNGZkMzQ3N2ItZWNjZi00ZWUzLThmN2QtNjhhZDcyMjYxNDc2IiwiYXVkIjoiZmFzdGFwaS11c2VyczphdXRoIiwiZXhwIjoxNTg3ODE4NDI5fQ.anO3JR8-WYCozZ4_2-PQ2Ov9O38RaLP2RAzQIiZhteM";
    curl \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $TOKEN" \
    -X GET \
    http://localhost:8000/users/me
    ```

=== "axios"
    ```ts
    const TOKEN = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiNGZkMzQ3N2ItZWNjZi00ZWUzLThmN2QtNjhhZDcyMjYxNDc2IiwiYXVkIjoiZmFzdGFwaS11c2VyczphdXRoIiwiZXhwIjoxNTg3ODE4NDI5fQ.anO3JR8-WYCozZ4_2-PQ2Ov9O38RaLP2RAzQIiZhteM';
    axios.get(
        'http://localhost:8000/users/me', {
        headers: {
            'Authorization': `Bearer ${TOKEN}`,
        },
    })
    .then((response) => console.log(response))
    .catch((error) => console.log(error));
    ```

### Ответ

Вы получите JSON-ответ, который будет выглядеть так:

```json
{
    "id": "4fd3477b-eccf-4ee3-8f7d-68ad72261476",
    "email": "king.arthur@camelot.bt",
    "is_active": true,
    "is_superuser": false
}
```

!!! tip
    Если вы используете одну из [вызываемых зависимостей](./current-user.md), чтобы защитить свой собственный конечный пункт, вы должны аутентифицироваться точно так же.

## 4. Обновление моего профиля

Мы также можем обновлять свой собственный профиль. Например, мы можем изменить свой пароль так.

### Запрос

=== "cURL"
    ``` bash
    curl \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $TOKEN" \
    -X PATCH \
    -d "{\"password\": \"lancelot\"}" \
    http://localhost:8000/users/me
    ```

=== "axios"
    ```ts
    axios.patch(
        'http://localhost:8000/users/me',
        {
            password: 'lancelot',
        },
        {
            headers: {
                'Authorization': `Bearer ${TOKEN}`,
            },
        },
    )
    .then((response) => console.log(response))
    .catch((error) => console.log(error));
    ```

### Ответ

Вы получите JSON-ответ, который будет выглядеть так:

```json
{
    "id": "4fd3477b-eccf-4ee3-8f7d-68ad72261476",
    "email": "king.arthur@camelot.bt",
    "is_active": true,
    "is_superuser": false
}
```

!!! info
    Еще раз, пользователь не может установить `is_active` или `is_superuser` самостоятельно. Это может сделать только суперпользователь, обновив пользователя с помощью PATCH.

## 5. Стать суперпользователем 🦸🏻‍♂️

Если вы хотите управлять пользователями вашего приложения, вам придется стать **суперпользователем**.

Первый суперпользователь может быть установлен только на **уровне базы данных**: откройте ее через интерфейс командной строки или графический интерфейс пользователя, найдите своего пользователя и установите свойство `is_superuser` в `true`.

### 5.1. Получение профиля любого пользователя

Теперь, когда вы суперпользователь, вы можете использовать мощь [маршрутов суперпользователя](./routes.md#superuser). Вы, например, можете получить профиль любого пользователя в базе данных по его идентификатору.

#### Запрос

=== "cURL"
    ``` bash
    curl \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $TOKEN" \
    -X GET \
    http://localhost:8000/users/4fd3477b-eccf-4ee3-8f7d-68ad72261476
    ```

=== "axios"
    ```ts
    axios.get(
        'http://localhost:8000/users/4fd3477b-eccf-4ee3-8f7d-68ad72261476', {
        headers: {
            'Authorization': `Bearer ${TOKEN}`,
        },
    })
    .then((response) => console.log(response))
    .catch((error) => console.log(error));
    ```

#### Ответ

Вы получите JSON-ответ, который будет выглядеть так:

```json
{
    "id": "4fd3477b-eccf-4ee3-8f7d-68ad72261476",
    "email": "king.arthur@camelot.bt",
    "is_active": true,
    "is_superuser": false
}
```

### 5.1. Обновление любого пользователя

Теперь мы можем обновить профиль любого пользователя. Например, мы можем повысить его до суперпользователя.

#### Запрос

=== "cURL"
    ``` bash
    curl \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $TOKEN" \
    -X PATCH \
     -d "{\"is_superuser\": true}" \
    http://localhost:8000/users/4fd3477b-eccf-4ee3-8f7d-68ad72261476
    ```

=== "axios"
    ```ts
    axios.patch(
        'http://localhost:8000/users/4fd3477b-eccf-4ee3-8f7d-68ad72261476',
        {
            is_superuser: true,
        },
        {
            headers: {
                'Authorization': `Bearer ${TOKEN}`,
            },
        },
    )
    .then((response) => console.log(response))
    .catch((error) => console.log(error));
    ```

#### Ответ

Вы получите JSON-ответ, который будет выглядеть так:

```json
{
    "id": "4fd3477b-eccf-4ee3-8f7d-68ad72261476",
    "email": "king.arthur@camelot.bt",
    "is_active": true,
    "is_superuser": true
}
```

### 5.2. Удаление любого пользователя

Наконец, мы можем удалить пользователя.

#### Запрос

=== "cURL"
    ``` bash
    curl \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $TOKEN" \
    -X DELETE \
    http://localhost:8000/users/4fd3477b-eccf-4ee3-8f7d-68ad72261476
    ```

=== "axios"
    ```ts
    axios.delete(
        'http://localhost:8000/users/4fd3477b-eccf-4ee3-8f7d-68ad72261476',
        {
            headers: {
                'Authorization': `Bearer ${TOKEN}`,
            },
        },
    )
    .then((response) => console.log(response))
    .catch((error) => console.log(error));
    ```

#### Ответ

Вы получите пустой ответ.

## 6. Выход

Мы также можем завершить сеанс.

### Запрос

=== "cURL"
    ``` bash
    curl \
    -H "Content-Type: application/json" \
    -H "Cookie: fastapiusersauth=$TOKEN" \
    -X POST \
    http://localhost:8000/auth/cookie/logout
    ```

=== "axios"
    ```ts
    axios.post('http://localhost:8000/auth/cookie/logout',
        null,
        {
            headers: {
                'Cookie': `fastapiusersauth=${TOKEN}`,
            },
        }
    )
    .then((response) => console.log(response))
    .catch((error) => console.log(error));
    ``

`

### Ответ

Вы получите пустой ответ.

## Заключение

Вот и все! Теперь у вас есть хороший обзор того, как вы можете управлять пользователями через API. Обязательно проверьте страницу [Маршруты](./routes.md), чтобы получить все детали по каждому из эндпоинтов.