---
title: Используем Docker для разработки и развертывания React-приложений. Часть 2.
date: "2019-07-22"
description: Готовим окружение для развертывания React-приложения с Docker.
---

###Это вторая часть, в которой мы разберемся с тем как подготовить окружение для развертывания Docker + React

<b>В [прошлой части](http://localhost:8000/react-ci-cd-1/) мы настраивали окружение для
разработки нашего приложения, пришло время подготовить окружения для деплоя.</b>

<b>Сегодня мы напишем второй Dockerfile для деплоя + настроим Nginx для раздачи статики.</b>

> [Репозиторий с итоговым кодом](https://github.com/RenatRysaev/dockerized-react/tree/react-ci-cd-2) для этой статьи

Первым делом давайте переименуем уже существующий Dockerfile -> Dockerfile.dev.  
Затем создадим второй докерфайл, назвав его Dockerfile.prod, который мы будем использовать для продакшена.

<b>Dockerfile.prod</b>
```
# Берем в качестве родительского образа node:8-apline и назовем эту ступень сборки "build-stage"
FROM node:8-alpine as build-stage
# Устанавливаем рабочую директорию
WORKDIR /app
# Копируем файлы package.json yarn.lock в рабочую директорию
COPY package.json yarn.lock /app/
# Устаналиваем зависимости
RUN yarn install
# Копируем исходники в рабочую директорию
COPY . /app/
# Собираем проект
RUN yarn build

# Вторая ступень сборки, поднимем nginx для раздачи статики
FROM nginx:stable-alpine
# Копируем билд из ступени сборки "build-stage" в директорию образа /usr/share/nginx/html
COPY --from=build-stage /app/build /usr/share/nginx/html
# Копируем конфиг nginx в директорию образа /etc/nginx/nginx.conf (Напишем его позже)
COPY nginx.conf /etc/nginx/nginx.conf
# Открываем 80 порт
EXPOSE 80
# Указываем команду, поднимающую nginx при запуске контейнера
CMD ["nginx", "-g", "daemon off;"]
```

### Напишем конфиг для Nginx

<b>nginx.conf</b>
```
# Автоматически определяем количество процессов
worker_processes auto;
events {
    # Устанавливаем максимальное кол-во соединений для одного рабочего процесса
    worker_connections 8000;
    # Указывем nginx, чтобы он пытался принять максимльное кол-во подключений
    multi_accept on;
}
http {
    # Указываем корректные заголовки для всех типов расширений
    include mime.types;
    default_type application/octet-stream;
    # Увеличиваем производительность
    sendfile on;
    # Сжимаем данные
    gzip on;
    gzip_comp_level 6;
    gzip_min_length 100;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    server {
        index index.html index.htm;
        # Слушаем 80 порт
        listen 80;
        # Директория из которой раздаем статику
        root /usr/share/nginx/html; 
        # Сначала пытаемся обработать запрос как к файлу или директории
        location / {
            try_files $uri $uri/ /index.html;
        }
    }
}
```

### Собираем образ и запускаем контейнер

Для того, чтобы собрать образ выполните:

```bash
docker build -t dockerized-react:prod -f Dockerfile.prod .
```

Для того, чтобы запустить контейнер и убедиться в том, что Nginx раздает статику выполните:

```bash
docker run -it -p 80:80 dockerized-react:prod
```

Откройте [localhost](http://localhost/)

Готово, теперь мы имеем контейнер с Nginx, который умеет раздавать сборку нашего приложения.

<b>В следующей статье мы будем разбираться с тем как с помощью Jenkins развернуть Docker-контейнер
на удаленном сервере.</b>
