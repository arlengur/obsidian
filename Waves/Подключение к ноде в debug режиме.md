## 1. Создание образа
 Из папки open-source-node выполнить 

```
sbt compile assembly // космпиляция проекта
sbt assembly // сборка проекта
docker login reg.web3tech.ru // авторизоваться в реесте
docker build -t wavesenterprise/my.test:v4 . // сборка образа (тег можно задать любой)```
```

## 2. Развертывание sandbox
- Создать папку sandbox и из нее выполнить команду
```
curl https://raw.githubusercontent.com/waves-enterprise/we-node/release-1.12/node/src/docker/docker-compose.yml -o docker-compose.yml // скачиваем docker-compose.yml
```

-  В файле docker-compose.yml
	- прописать созданный образ (для всех нод)
	```
	image: wavesenterprise/my.test:v4
	```
	- Для подключения к ноде в дебагрежиме добавить следующие параметры
	```
	ports:
	  - "5005:5000"
	environment:
	      - JAVA_OPTS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5000
	```

При remote подключении подключаемся к локалхосту но в терминале выполняем:
```
kubectl port-forward -n ibudget-dev pod/node-0 5005:5005
```

-  Для настройки конфигурации из папки sandbox выполнить
```
docker run --rm -ti -v $(pwd):/config-manager/output wavesenterprise/config-manager:latest
```
- Проверить, что нода содержит корректную настройку для api key (./configs/nodes/node-0/node.conf)
```
node {
	api {
		auth {
			type = "api-key"
			api-key-hash = "5M7C14rf3TAaWscd8fHvU6Kqo97iJFpvFwyQ3Q6vfztS"
			privacy-api-key-hash = "5M7C14rf3TAaWscd8fHvU6Kqo97iJFpvFwyQ3Q6vfztS"
	    }
	}
}`
```

-  Запуск sandbox
```
docker-compose up -d
```

-  Остановка sandbox
```
docker-compose down
```

Замечание: Если при запуске блокчейн ноды не стартуют, то можно попробовать почистить тома (docker volume prune - удалит все неиспользуемые тома)

## 3. Подключение к sandbox в debug режиме

- В intelliJ Idea создать новую конфигурацию: Run - Edit Configurations... - Remote JVM Debug 
    - Name - имя конфигурации
    - Host - адрес машины на которой запущена sandbox (127.0.0.1)
    - Port - порт по которому подключаемся (5005)
    - Command line arguments for remote JVM (-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005)

Замечание: Сваггер стартует по адресу [http://localhost:6862/api-docs/index.html](http://localhost:6862/api-docs/index.html)