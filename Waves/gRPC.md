https://confluence.web3tech.ru/pages/viewpage.action?pageId=1836186
- Открыть Postman
- Создать Workspace
- Создать gRPC request, в нем
	- В строке адреса прописать адрес сттенда (например: grpc://localhost:6865)
	- Импортировать *.proto файлы (их можно найти в репозитории [https://gitlab.web3tech.ru/development/we/qa/we-autotests/-/tree/release-1.13.0/](https://gitlab.web3tech.ru/development/we/qa/we-autotests/-/tree/release-1.13.0/ "https://gitlab.web3tech.ru/development/we/qa/we-autotests/-/tree/release-1.13.0/")), если не хватает зависимостей - добавить необходимые папки или всю we_proto
	- Указать имя сервиса (SubscribeOn)
	- Авторизация: API Key (key: X-API-Key, value: vostok)
	- Заполнить Message
```
{
"genesis_block": {}
}
```
или 
```
"current_event": {}
```
или
```
{
	"block_signature": {
		"last_block_signature": {
			"value": "G4gTl/5fA2g2YAFCjCGu+tXJVqvQCLNM8CxzT6Nfc3KSRg3egAY8Mb4df5tufIf9Tv2xfCPQQ5m7X4MoPBvnBg=="
		}
	}
}
```
