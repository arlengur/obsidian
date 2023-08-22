- Высота ноды
```
http://localhost:6862/blocks/height
```
- Ручки
```
http://localhost:6862/api-docs/index.html
```
# Как поменять версию библиотеки open-source-we-core

- после пуша ветки в gitlab ci, пушим либу в наш nexus, и после этого можно подключать к ноде обновленную we-core
	- /corporate-node/open-source-node/project/build.properties
	- /corporate-node/project/build.properties

тут указывается версия we-core, как правило она одинаковая для open-source и corporate

Версию можно узнать после пуша в CI, в логах
https://gitlab.web3tech.ru/development/we/node/open-source-we-core/-/jobs/478854

Пример: published ... 1.12.3-54-WE-8663-add-grpc-confidentialExecutedTxByExecutableTxId-core-ed0a2a3-SNAPSHOT ...

# Обновить образ на стенде

- перейти в Pipelines с нужным коммитом ([https://gitlab.web3tech.ru/development/we/node/corporate-node/-/pipelines/106467](https://gitlab.web3tech.ru/development/we/node/corporate-node/-/pipelines/106467) )

- в разделе Pipeline есть publish-docker-images, если по не му нажать то в логе можно найти нужную строку: Successfully tagged reg.web3tech.ru/development/we/node/corporate-node:WE-8554-post-metrics-404-instead-400

- на тестовом стенде: https://gitlab.web3tech.ru/devops/argocd/ovh/we-dev/-/blob/master/overlays/clinton/nodes.yaml

- пометь ссылку на тег/образ: containers - image: reg.web3tech.ru/development/we/node/corporate-node:WE-8554-post-metrics-404-instead-400

