# Базовые запросы к MongoDB

## Задание

- [x] установить MongoDB в ВМ В GCP
- [x] заполнить данными в какой-либо предметной области, например интернет-магазин;
- [ ] написать несколько запросов на выборку, обновление и удаление данных

## Описание выполнения

С помощью комманд:

```js
use mflix

db.createCollection("comments")
db.createCollection("movies")
db.createCollection("sessions")
db.createCollection("theaters")
db.createCollection("users")
```

, я создал базу данный **mflix**, и в ней создал коллекции: **comments**, **movies**, **sessions**, **threaters**, **users**. И из репозитория <https://github.com/neelabalan/mongodb-sample-dataset/tree/main/sample_mflix> я скачал данные.
