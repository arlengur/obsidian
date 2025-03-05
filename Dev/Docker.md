
| Описание                                             | Комана                                                                    |
| ---------------------------------------------------- | ------------------------------------------------------------------------- |
| Версия                                               | docker -v \| docker version                                               |
| Скопировать удаленный образ                          | docker pull node                                                          |
|                                                      |                                                                           |
| Запустить образ                                      | docker run <image_id>                                                     |
| Запустить определенную версию образа                 | docker run <image_id>:latest                                              |
| Запустить образ как демон                            | docker run -d <image_id>                                                  |
| Запустить образ на порту с именем name               | docker run -p <local_port>:<container_port> -- name image_name <image_id> |
| Запустить образ и удалить сразу как будет остановлен | docker run -d --name name --rm <image_id>                                 |
| Запустить образ в интерактивном режиме               | docker run -it <image_id>                                                 |
| Подключиться к образу в интерактивном режиме         | docker exec -it <image_id> bash                                           |
|                                                      |                                                                           |
| Запустить контейнер                                  | docker start <container_id>                                               |
| Остановить контейнер                                 | docker stop <container_id>                                                |
| Удалить контейнер                                    | docker rm <container_id>                                                  |
| Удалить все контейнеры                               | docker container prune -f                                                 |
| Удалить образ                                        | docker rmi image                                                          |
| Удалить все образы                                   | docker image prune                                                        |
| Создать образ                                        | docker build .                                                            |
| Посмотреть список образов                            | docker image ls или docker images                                         |
| Показать все контейнеры                              | docker ps -a                                                              |
| Запустить контейнер в интерактивном режиме           | docker run -it node                                                       |
| Выйти из интерактивного режима                       | exit                                                                      |
| Логи контейнера                                      | docker logs <container_id>                                                |
| Дать имя образу                                      | docker build -t logsapp .                                                 |
| Дать имя и версию образу                             | docker build -t logsapp:1 .                                               |

Image - шаблон из которого создается контейнер, они только для чтения
hub.docker.com - репозиторий образов

Между собой контейнеры общаются при помощи внутренних портов.
# Dockerfile
```
FROM python -- имя образа

WORKDIR /app -- рабочая директория в образе

COPY . . -- что в нее скопировать (то есть копируем из корня нашего проекта в рабочую директорию)

RUN npm install -- выполняется при сборке образа (1 раз)

EXPOSE 3000 -- используемый порт

CMD ["python", "index.py"] -- выполняется каждый раз при запуске образа
```

Залить в docker hub:
```
docker login
docker push <login>/logsapp:latest
```

Файл для исключения файлов из образа докера `.dockerignore`

## Volume
Хранение вне контейнера. Добавим в докер файл строку
`VOLUME ["/app/data"]`

Volume могут быть анонимные и именованные. Чтобы создать именованный Volume
`docker run -d -p 3000:4200 -v logs:/app/data --rm --name logsapp logsapp:env`

Чтобы посмотреть созданные Volume
`docker volume ls`

Удалить Volume
```
docker volume rm logs
docker volume prune
```

А чтобы изменения в коде отображались на Volume:
`docker run -d -p 3000:4200 -v "<path_to_project_dir>:/app" -v /app/node_modules -v logs:/app/data --rm --name logsapp logsapp:env`

## Env
В докер файле можно определить переменную:
`ENV PORT 4200` и потом ссылаться на нее `EXPOSE $PORT`

Задать свою переменную при запуске контейнера:
`docker run -d -p 3000:80 -e PORT=80 --rm --name logsapp logsapp:env`

Можно создать отдельный файл с переменными в папке config .env:
`PORT=4200`

тогда при запуске надо будет указать его так:
`docker run -d -p 3000:4200 -env-file ./config/.env --rm --name logsapp logsapp:env`

## Makefile
Можно создать алиасы для выполняемых команд, для этого нужна программа make и создать в проекте файл Makefile:
run:
    docker run -d -p 3000:80 -e PORT=80 --rm --name logsapp logsapp:env
stop:
    docker stop logsapp

и потом запускать можно так:
make run
make stop

# Docker-compose

```
services: // описываем контейнеры
 mysql:
  image: mysql:8.0 // образ с докер хаба
  container_name: mysql8 // имя контейнера
  dependence_on: // после какого контейнера загружать данный контейнер
   - mysql
  restart: no // политика перезапуска
  env_file: .env // файл с переменными окружения
  environment: // переменные которые не поместились в файле
   - WORDPRESS_DB_HOST=mysql:3306
  ports: // прокидываем порты первый - хост машина, второй - внутри контейнера
   - 80:80
  volumes: // подключаем именованные тома
   - dbfile:/var/lib/mysql // подключить dbfile по пути /var/lib/mysql
  command: '--default-authentication-plugin=mysql_native_password' // команда которая выполнится
  networks: // принадлежность к сети
   - app

volumes: // что где храним на диске
 www-html:
 dbfile:

networks: // описываем сеть
 app:
```

Команды:
```
docker-compose up -d // запуск в фоновом режиме
docker-compose up -d nginx // запуск только указанного контейнера
docker-compose up --force-recreate --no-deps -d nginx // перезапуск только указанного контейнера

docker-compose ps // запущенные контейнеры

docker-compose down // остановка контейнера

docker compose logs node-0 -f // gросмотр логов ноды
```
