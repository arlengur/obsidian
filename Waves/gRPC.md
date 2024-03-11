https://confluence.web3tech.ru/pages/viewpage.action?pageId=1836186
- Открыть Postman
- Создать Workspace
- Создать gRPC request, в нем
	- В строке адреса прописать адрес сттенда (например: grpc://localhost:6865)
	- Импортировать *.proto файл (их можно найти в репозитории [https://gitlab.web3tech.ru/development/we/qa/we-autotests/-/tree/release-1.13.0/](https://gitlab.web3tech.ru/development/we/qa/we-autotests/-/tree/release-1.13.0/ "https://gitlab.web3tech.ru/development/we/qa/we-autotests/-/tree/release-1.13.0/")), если не хватает зависимостей - добавить необходимые папки из проекта open-source-we-core (в зависимости от версии системы прото файлы меняются поэтому перед импортом необходимо пересобрать проект):
		- /Users/agalin/repo/proto/1.13/transaction-protobuf/src/main/protobuf
		- /Users/agalin/repo/proto/1.13/grpc-protobuf/src/main/protobuf
	- Выбрать Import as API
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

Создание транзакции
- Подписать транзакцию через ручку:
```json
{
  "type": 103,
  "version": 2,
  "sender": "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",
  "password": "sato",
  "image": "vostok-sc/grpc-contract-example:v3.0",
  "contractName": "grpc-contract-example-v3",
  "imageHash": "a5d55175718e0de512847d199459295bc5dc22aa9090a64ecd551586eba671cc",
  "params": [
    {
      "type": "string",
      "key": "accounts",
      "value": "[{\"accountNumber\":\"1119810\"}]"
    }
  ],
  "fee": 100000000,
  "timestamp": 1696850364002,
  "feeAssetId": null
}
```

- Вставить в grpc запрос значение полей id и proofs из результата
	- где id, sender_public_key, proofs, sender_address должны быть в base64
```json
{
"create_contract_transaction": {
	"id": "+JMBBgTpLkMmfuE9vpOaw2O3Ze/cHA2QQrvnSQVwO24=",
	"sender_public_key": "JyQ80GW0lmVj81n6zcF+3vGnkaj3xhw7BJ/vigr41Rw=",
	"image": "vostok-sc/grpc-contract-example:v3.0",
	"image_hash": "a5d55175718e0de512847d199459295bc5dc22aa9090a64ecd551586eba671cc",
	"contract_name": "grpc-contract-example-v3",
	"params": [{
		"string_value": "[{\"accountNumber\":\"1119810\"}]",
		"key": "accounts"
	}],
	"fee": "100000000",
	"timestamp": "1696850364002",
	"atomic_badge": {},
	"validation_policy": {
		"any": {}
	},
	"api_version": {
		"major_version": 1,
		"minor_version": 0
	},
	"payments": [],
	"proofs": [
	"LUBG4xeg7ve7jDu/WBGQ1DSpl+q61pi5W5yh07G07jzrZL9zS7HB438yukXN04CPQameyaXFnbJBD9VDA8JbDw=="
	],
	"sender_address": "AUtoYXJzDweDLqNy82bZihyYbtIaUNNdUqU="
},
"version": 2
}
```