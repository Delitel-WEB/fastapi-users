# Установка

Вы можете добавить **FastAPI Users** в свой проект FastAPI всего лишь в несколько простых шагов. Прежде всего, установите зависимость:

## С поддержкой SQLAlchemy

```sh
pip install 'fastapi-users[sqlalchemy]'
```

## С поддержкой Beanie

```sh
pip install 'fastapi-users[beanie]'
```

## С поддержкой аутентификации через бэкенд Redis

Информацию по установке с правильной поддержкой базы данных можно найти в разделе [Redis](configuration/authentication/strategies/redis.md).

## С поддержкой OAuth2

Информацию по установке с правильной поддержкой базы данных можно найти в разделе [OAuth2](configuration/oauth.md).

---

Вот и всё! В следующем разделе мы рассмотрим [обзор](./configuration/overview.md) того, как это работает.