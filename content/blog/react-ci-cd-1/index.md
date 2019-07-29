---
title: Используем Docker для разработки и развертывания React-приложений. Часть 1.
date: "2019-07-21"
description: Готовим окружение для разработки React-приложения с Docker.
---

##Я планирую серию статей, в которых постараюсь раскрыть следующие темы:
1) [<b>Docker для разработки React-приложения</b>](https://rysaev.dev/react-ci-cd-1/)
2) [<b>Docker для развертывания React-приложения + Nginx для раздачи статики</b>](https://rysaev.dev/react-ci-cd-2/)
3) <b>CI/CD с использованием Jenkins</b>

###Это первая часть, в которой мы разберемся с тем как настроить окружение для разработки с Docker + React

### Вступление

Все началось с того, что однажды на рабочем проекте потребовалось обновить зависимости.
После долгих мучений я наконец-то запушил свой коммит в репозиторий и отправился к нашему девопсу,
чтобы попросить задеплоить сборку из jenkins на тестовый сервер. Ну и как вы уже, наверное, догадываетесь девопсу этого сделать
не удалось, потому что на сервере с jenkins стояла node.js другой версии и закешировались старые версии библиотек.
Я бы скорее всего решил эту проблему, но на следующий рабочий день меня переводили на другой проект.

Ну собственно я к чему, используя docker, мы точно бы не имели этих проблем, и вот почему:
1) Единое окружение локально и на сервере
2) Воспроизводимость

Поэтому я решил немного раскурить Docker и поделиться полезной информацией о том
как его использовать для разработки и деплоя.  

> [Репозиторий с итоговым кодом](https://github.com/RenatRysaev/dockerized-react/tree/react-ci-cd-1) для этой статьи

<b>Предпологается что на вашей машине уже установлено:</b>
1) [Docker](https://www.docker.com/) v18.09.2
2) [Create React App](https://github.com/facebook/create-react-app) v3.0.1

###Создание проекта

```bash
create-react-app dockerized-react
cd dockerized-react
```

###Напишем наш Dockerfile для создания образа

<b>Dockerfile</b>
```dockerfile
# Берем node.js v8 в качестве родительского образа
FROM node:8
# Устанавливаем рабочую директорию
WORKDIR /app
# Копируем файлы yarn.lock и package.json в рабочую директорию
COPY package.json yarn.lock /app/
# Устанавливаем зависимости
RUN yarn install
# Открываем порт 3000
EXPOSE 3000
# Запускаем React-приложение после запуска контейнера
CMD ["yarn", "start"]
```

### Добавим в .dockerignore директории/файлы, которые не должны попасть в образ

<b>.dockerignore</b>
```
.git
node_modules
build
```

### Создание образа

```bash
docker build -t dockerized-app:dev .
```

### Запуск контейнера

```bash
docker run -it -v ${PWD}:/app -v /app/node_modules -p 3001:3000 -e CHOKIDAR_USEPOLLING=true dockerized-app:dev
```

Для Windows

```bash
docker run -it -v %cd%:/app -v /app/node_modules -p 3001:3000 -e CHOKIDAR_USEPOLLING=true dockerized-app
```

<b>Давайте разберемся для чего и какие аргументы мы передаем</b>

1) "-it" - интерактивный режим
2) "-v ${PWD}:/app" - монтируем текущую рабочую директорию в директорию /app docker-контейнера
3) "-v /app/node_modules" - используем node_modules из контейнера
4) "-p" - сопоставляем порт docker-контейнера порту на хост системе
5) "-e CHOKIDAR_USEPOLLING=true" - устанавливаем переменную окружения для принудительного слежения webpack за нашим томом.

Осталось открыть приложение в браузере [http://localhost:3001/](http://localhost:3001/)

Готово, теперь webpack отслеживает изменения файлов и изменив код в каком-либо файле мы сможем увидеть результат в браузере.

<b>В следующей статье мы будем разбираться с тем подготовить окружение для развертывания Docker + React</b>