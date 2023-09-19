
- Высота ноды
```
curl http://localhost:6862/blocks/height
```

для стенда
```
curl -X GET "https://clinton.welocal.dev/node-0/blocks/height" -H "accept: application/json" -H "X-API-Key: vostok"
```

```
curl -X GET "localhost:6862/blocks/seq/865/874" -H "accept: application/json" -H "X-API-Key: vostok"
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

Пример: `published ... 1.12.3-54-WE-8663-add-grpc-confidentialExecutedTxByExecutableTxId-core-ed0a2a3-SNAPSHOT ...`

# CryptoPro
- JCSP https://cryptopro.ru/products/csp/downloads#latest_csp50r3_jcsp
- JCP https://cryptopro.ru/products/csp/jcp/downloads

# Waves repo
- https://artifacts.wavesenterprise.com/

# Гитлаб репо
- corporate https://gitlab.web3tech.ru/development/we/node/corporate-node/-/pipelines
- open source https://gitlab.web3tech.ru/development/we/node/open-source-node
- core https://gitlab.web3tech.ru/development/we/node/open-source-we-core/-/pipelines
- legacy (1.8.4 & 1.9) https://gitlab.web3tech.ru/development/we/node/legacy-vostok
- legacy core https://gitlab.web3tech.ru/development/we/node/we-core/

# Создать учетную запись
- https://sato.welocal.dev/auth/welcome
- Роли пользователя хранятся в бд auth_service там табличка users_roles_roles

# Выпуск кандидата
```
git tag v1.12.4-RC1 // в corporate-node
git subtree push --prefix open-source-node open-source-origin open-source-repo-branch
открыть мр в open-source
сделать мерж
ставим тег v1.12.4-RC1 в open-source
```

# Выпуск релиза
```
git commit --allow-empty -m "Release 1.12.4"  // пустой коммит с названием релиза
git tag v1.12.4
git push origin release-1.12
git push origin v1.12.4

тег проставляется в гитлаб
```

Если изменения касаются open-source версии, то нужно сделать subtree push

# Легаси версия core
- Сделать мерж в релизную ветку
- Поставить в гиттлабе тег `v1.8.4-RC1`
- Найти созданную библиотеку в https://artifacts.wavesenterprise.com/#browse/browse:we-snapshots

# Легаси версия node
- Сделать мерж в релизную ветку
- Поставить в гиттлабе тег `v1.8.6-RC1`
- В publish-docker-images найти строку `Successfully tagged reg.web3tech.ru/development/we/node/legacy-vostok-public:v1.8.6-RC1` ссылкой будет `reg.web3tech.ru/development/we/node/legacy-vostok`