Asset - токен или монета
Account - кошелек пользователя с балансом

TimeSheet:
https://confluence.web3tech.ru/pages/viewpage.action?pageId=27328837

Документация:
https://confluence.web3tech.ru/pages/viewpage.action?spaceKey=VO&title=Circuit+Breaker

Заявки на отпуска, дэй-офф, ДМС и пр. пишем сюда:
https://jira.web3tech.ru/servicedesk/customer/portals
Чтобы воспользоваться сервисом, обязательно нужно быть авторизованным на [https://jira.web3tech.ru/](https://jira.web3tech.ru/ "https://jira.web3tech.ru/") По всем возникающим вопросам пишите нашему сисадмину Косте Мялкину @Konstantin Myalkin

sha256 = 32bytes

Валидация комиссии fee зависит от настройки feature-check-blocks-period и работает когда высота ноды выше значения указанного там.

Про `feature-check-blocks-period`
https://docs.wavesenterprise.com/en/latest/description/soft-forks.html?highlight=features

Реджистри
```
default-registry-domain = "registry.wavesenterprise.com/waves-enterprise-public"
default-registry-domain = "registry.wvservices.com"
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

# Waves repo (nexus)
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

https://repo1.maven.org/maven2/com/wavesenterprise/we-crypto/1.8.5/we-crypto-1.8.5.pom
https://artifacts.wavesenterprise.com/repository/we-releases/com/wavesenterprise/we-crypto/1.8.4-RC1/we-crypto-1.8.4…