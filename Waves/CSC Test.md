1. Добавить в vault конфиг `confidential-contracts-api-key-hash` в node.conf (hash можно взять такой же, как для api-key-hash или privacy-api-key-hash)
```json
api {
	...
	auth {
		type = "api-key"
		...
		confidential-contracts-api-key-hash = "5M7C14rf3TAaWscd8fHvU6Kqo97iJFpvFwyQ3Q6vfztS"
	}
}
```
2. Открыть http://localhost:6862/api-docs/index.html
3. На странице сваггера в строке где задается используемый json ввести такой адрес http://localhost:6862/api-docs/base-open-api.json
4. Дернуть ручку POST​ /transactions/signAndBroadcast которая подписывает и отправляет транзакцию (из ответа надо взять id и timestamp):
```json
{  
	"apiVersion": "1.0",  
	"contractName": "grpc-increment-contract-csc here meet",  
	"fee": 100000000,  
	"feeAssetId": null,  
	"groupOwners": [  
		"3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",  
		"3JabCA2bX4qEnPn8ceMpNLeJRPadjjBnzPa",  
		"3JLNysbvrhUouzupPUES1tPux3z5MJ19FXS"  
	],  
	"groupParticipants": [  
		"3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",  
		"3JabCA2bX4qEnPn8ceMpNLeJRPadjjBnzPa",  
		"3JLNysbvrhUouzupPUES1tPux3z5MJ19FXS"  
	],  
	"image": "reg.web3tech.ru/development/we/node/corporate-node/grpc-increment-contract-csc:latest",  
	"imageHash": "7a8a3a69825d47e6fdf376257bd822c70ffcdbd7e52636aeb1cf6812ac2af610",  
	"isConfidential": true,  
	"params": [],  
	"password": "sato",  
	"payments": [],  
	"sender": "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",  
	"type": 103,  
	"validationPolicy": {  
		"type": "any"  
	},  
	"version": 6  
}
````
5. В метод авторизации `Confidential-Contracts-API-Key (apiKey)` добавить свой ключ, который в vault указали
6. Дернуть ручку POST​ /confidential-contracts​/call которая вызывает контракт
	```json
{  
	"sender": "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",  
	"password": "sato",  
	"contractId": "<id from /transactions/signAndBroadcast>",  
	"contractVersion": 1,  
	"params": [
		{  
			"key": "COUNTERS_COUNT",  
			"type": "integer",  
			"value": 2  
		}  
	],  
	"timestamp": "<timestamp>",  
	"fee": 10000000,  
	"certificatesBytes": []  
}
```
7. gRPC version
```json
{  
	"sender": "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",  
	"password": {  
		"value": "sato"  
	},  
	"contract_id": "GNEsBi2HzuUBezczBBiQM6mttdWKv3NhGt5joURZgb3W",  
	"contract_version": 1,  
	"params": [  
		{  
			"key": "COUNTERS_COUNT",  
			"int_value": 2  
		}  
	],  
	"timestamp": 1692782534361,  
	"fee": 10000000,  
	"certificatesBytes": []  
}
```
8. Дернуть ручку POST​ `/confidential-contracts/tx/{executable-tx-id}` которая возвращает тот же результат
9. gRPC version
```json
{
	"transactionId": "DtZE8yFW8twQZGYW4sSRFu6TYYJi51fWCm9vNoPgDAkY"
}
```
# Примечание
- contractName - имя контракта (мб любым)
- sender - отправитель
- fee - комиссия
- params -  метод, который вызывается при деплое контракта
- password - пароль ключевой пары на ноде
- type 103 - создание контракта
- type 104 - вызов контракта
- type 107 - обновление контракта
- type 111 - добавление ноды
- Образ КСК: reg.web3tech.ru/development/we/node/corporate-node:WE-8149-confidential-smart-contracts

- В 103 транзакции (создание контракта) можно вызывать метод @ContractInit (их мб несколько), это указывается в параметрах транзакции
```json
для
@ContractInit
fun create()

будет:
'params': [
  {
    "type": "string",
    "key": "action",
    "value": "create"
  }
]

для
@ContractInit
fun create(@InvokeParam(name='x') x: String)

будет:
'params': [
  {
    "type": "string",
    "key": "action",
    "value": "create"
  },
  {
    "type": "string",
    "key": "x",
    "value": "x value"
  }
]
```

- В 104 транзакции (вызов контракта) можно вызывать метод @ContractAction
- Для загрузки образа из докер-репозитория в ноде должна быть настройка
```json
remote-registries = [
	{
		username = "robot$harbor_vostok-sc_robot"
		password = "b52NxkcjmqoQBQTw4JsDBvhZlhGcul3E"
		domain = "registry.wvservices.com"
	},
	{
		username = "webuild"
		password = "glpat-6ePd9736BG2wcr3cd8ky"
		domain = "reg.web3tech.ru"
	}
]
```
- Для создания контракта необходимо назначить валидаторов, для этого нужно дать ноде право "contract_validator", но после этого нужно будет переподписать генезис или выполнить транзакцию
```json
  {  
	"type": 102,  
	"sender": "3NkZd8Xd4KsuPiNVsuphRNCZE3SqJycqv8d",  
	"target": "3NkZd8Xd4KsuPiNVsuphRNCZE3SqJycqv8d",  
	"opType": "add",  
	"role": "contract_validator",  
	"password": "<password>",  
	"fee": 1000000,  
	"version": 2  
}
```
- Список необходимых фич в настройках ноды (их описание [https://docs.wavesenterprise.com/en/latest/description/soft-forks.html?highlight=features](https://docs.wavesenterprise.com/en/latest/description/soft-forks.html?highlight=features "https://docs.wavesenterprise.com/en/latest/description/soft-forks.html?highlight=features"))
```json
2 = 0
3 = 0
4 = 0
5 = 0
6 = 0
7 = 0
9 = 0
10 = 0
100 = 0
101 = 0
119 = 0
120 = 0
130 = 0
140 = 0
160 = 0
162 = 0
173 = 0
180 = 0
1120 = 0
1122 = 0
1123 = 0
1130 = 0
```