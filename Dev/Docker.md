Запуск:
`docker-compose up -d`

Остановка:
`docker-compose down`

Просмотр логов ноды:
`docker compose logs node-0 -f`

Скопировать удаленный образ:
`docker pull node`

Запустить образ: 
`docker run <image_id>`

```

docker version
docker -v

Пример: 
FROM python -- имя образа

WORKDIR /app -- рабочая директория в образе

COPY . . -- что в нее скопировать (то есть копируем из корня нашего проекта в рабочую директорию)

CMD ["python", "index.py"] -- какую команду выполнить


Создать образ: 
docker build .

Посмотреть список созданных образов: 
docker image ls
docker images

Image - шаблон из которого создается контейнер, они только для чтения
hub.docker.com - репозиторий образов

Показать документацию по команде:
docker ps --help

Показать все контейнеры:
docker ps -a

Запустить контейнер в интерактивном режиме:
docker run -it node (можно выполнить команду: 1+1 или process.version)

Выйти из интерактивного режима:
.exit

Удалить контейнер:
docker rm <container_id>

Удалить все контейнеры:
docker container prune
docker container prune -f - без вопроса подтверждения

Пример: 
FROM node

WORKDIR /app 

COPY . . 

RUN npm install -- выполняется при сборке образа (1 раз)

EXPOSE 3000 -- используемый порт

CMD ["node", "app.js"] -- выполняется каждый раз при запуске образа


Запустить образ на порту (run работает только с контйнерами):
docker run -p <local_port>:<container_port> <IMAGE_ID>

Запустить образ как демон
docker run -d -p <local_port>:<container_port> <IMAGE_ID>

Запустить контейнер:
docker start <container_id | name>

Остановить контейнер:
docker stop <container_id | name>

Присоединяется к запущенному контейнеру:
docker attach CONTAINER_ID

Логи контейнера:
docker logs CONTAINER_ID

Запустить образ и дать ему имя:
docker run -d -p 3000:3000 -—name logsapp IMAGE_ID

Запустить образ и удалить его сразу как он будет остановлен:
docker run -d -p 3000:3000 -—name logsapp --rm IMAGE_ID

Удалить образы:
docker rmi IMAGE IMAGE

Удалить все образы:
docker image prune

Дать имя logsapp  образу:
docker build -t logsapp .

Дать имя logsapp и версию образу:
docker build -t logsapp:1 .

Запустить определенную версию образа:
docker run -d -p 3000:3000 --name logsapp logsapp:1

Создать образ на основе существующего образ:
docker tag logsapp agalin/logsapp

Залить в docker hub:
docker login
docker push <login>/logsapp:latest

Файл для исключения файлов из образа докера:
.dockerignore:
node_modules
.git
Dokerfile

В докер файле можно определить переменную:
ENV PORT 4200

и потом ссылаться на нее:
EXPOSE $PORT

Как задать свою переменную при запуске контейнера:
docker run -d -p 3000:80 -e PORT=80 --rm --name logsapp logsapp:env

Можно создать отдельный файл с переменными в папке config .env:
PORT=4200

тогда при запуске надо будет указать его так:
docker run -d -p 3000:4200 -env-file ./config/.env --rm --name logsapp logsapp:env

Можно создать алиасы для выполняемых команд, для этого нужна программа make и создать в проекте файл Makefile:
run:
    docker run -d -p 3000:80 -e PORT=80 --rm --name logsapp logsapp:env
stop:
    docker stop logsapp

и потом запускать можно так:
make run
make stop

Хранение вне контейнера. Добавим в докер файл строку:
VOLUME ["/app/data"]


Volume могут быть анонимные и именованные. Чтобы создать именованный Volume:
docker run -d -p 3000:4200 -v logs:/app/data --rm --name logsapp logsapp:env

Чтобы посмотреть созданные Volume:
docker volume ls


Удалить Volume:
docker volume rm logs
docker volume prune

А чтобы изменения в коде отображались на Volume:
docker run -d -p 3000:4200 -v "<path_to_project_dir>:/app" -v /app/node_modules -v logs:/app/data --rm --name logsapp logsapp:env
```

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
docker-compose start|stop // запуск и остановка контейнера
```
