# POST `/pki/verify`
Ошибки:
- `Authorization error. Missing authorization` metadata говорит о том что нужно авторизоваться
# POST `transactions/signAndBroadcast`

## Перевод (4)
Адрес sender должен совпадать с адресом ноды
```json
{
  "type": 4,
  "version": 2,
  "sender": "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",
  "password": "sato",
  "recipient": "3JabCA2bX4qEnPn8ceMpNLeJRPadjjBnzPa",
  "amount": 10,
  "fee": 0
}

curl -X GET "localhost:6862/addresses/balance/3JabCA2bX4qEnPn8ceMpNLeJRPadjjBnzPa" -H "accept: application/json" -H "X-API-Key: vostok"
```
## Создать контракт (103)
https://docs.wavesenterprise.com/ru/latest/description/transactions/tx-list.html#createcontract-transaction
```json
// version 2
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
  "timestamp": 1651487626477,
  "feeAssetId": null
}

// version 4|5
{
  "type": 103,
  "version": 4,
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
  "timestamp": 1696411953991,
  "feeAssetId": null,
  "atomicBadge": null,
  "validationPolicy": {
    "type": "any"
  },
  "apiVersion": "1.0"
}

// version 6
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
```
## Обновить список участников (107)
```json
{  
"apiVersion": "1.0",  
"image" : "reg.web3tech.ru/development/we/node/corporate-node/grpc-increment-contract-csc:latest",  
"sender" : "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",  
"password": "sato",  
"fee" : 100000000,  
"contractId" : "5EfU85LzJFmFwjn1EqReBzJtZCRzgHMcKmAXvnm9S5pJ",  
"imageHash" : "7a8a3a69825d47e6fdf376257bd822c70ffcdbd7e52636aeb1cf6812ac2af610",  
"type" : 107,  
"version" : 5,  
"atomicBadge" : null,  
"groupOwners": [  
"3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",  
"3JabCA2bX4qEnPn8ceMpNLeJRPadjjBnzPa",  
"3JLNysbvrhUouzupPUES1tPux3z5MJ19FXS",  
"3JVKPPBCNXxzdh8t9SMmk7VRGeBx6Ebm6wX"  
],  
"groupParticipants": [  
"3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",  
"3JLNysbvrhUouzupPUES1tPux3z5MJ19FXS",  
"3JVKPPBCNXxzdh8t9SMmk7VRGeBx6Ebm6wX"  
],  
"validationPolicy": {  
"type": "any"  
}  
}
```
## Добавить ноду (111)
```json
{ 
"type": 111, "opType": "add", "sender":"3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv", 
"password": "", 
"targetPubKey": "DZf6SFs6osmPq3q16M1ra17rX8HW8Hs49yWFGUsu8EiU", "nodeName": "node-3", 
"fee": 1100000 
}
```
## Дать роль ноде (102)
```json
{
  "type": 102,
  "sender": "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",
  "password": "sato",
  "senderPublicKey": "3dnx34pAHDXKhavFRbafMuMo9oC7t7qjNT9UiuFTVBLf",
  "fee": 0,
  "target": "3JVKPPBCNXxzdh8t9SMmk7VRGeBx6Ebm6wX",
  "opType": "add",
  "role": "contract_validator",
  "dueTimestamp": null,
  "version": 2
}
```
## Вызвать контракт (104)
```json
{
  "type": 104,
  "version": 2,
  "contractId": "7HYWWYQgrqaqts8bGrjnFncbSmJdBFGwbyxSeizgXWrU",
  "fee": 10000000,
  "sender": "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",
  "password": "sato",
  "params": [
    {
      "type": "integer",
      "key": "a",
      "value": 1
    }
  ],
  "contractVersion": 2
}
```

# GET /contracts/{contractId} 
получить состояние контракта

# GET /contracts/info/{contractId}
получить информацию о контракте

# GET ​/contracts​/executed-tx-for​/{id}
результат выполнения транзакции

# GET /confidential-contracts/tx/{executable-tx-id}
получить состояние КСК

# POST /confidential-contracts/call

# GET ​/confidential-contracts/{contractId}
получить состояние конфиденциального контракта
```
curl -X GET "localhost:6862/confidential-contracts/GdbHkuTkarVUQ2D7vqKHNMYJy4i7RU6ubbU8rVKGMZ7F" -H "accept: application/json" -H "X-API-Key: vostok"
```
## Call запрос (104)
```json
{  
"sender": "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv",  
"password": "sato",  
"contractId": "9ZbVSzGWmWWDXDrzQnxcKuM4Mv5XMqXe75xcoRZxM3aX",  
"contractVersion": 1,  
"params": [],  
"fee": 10000000,  
"certificatesBytes": [],  
"timestamp": 1693552226536  
}
```
# GET /blocks/seq/97/101
вывести блоки с 97 по 101
# GET /blocks/ at/97
вывести блок 97
```
curl -X GET "localhost:6862/blocks/seq/865/874" -H "accept: application/json" -H "X-API-Key: vostok"
```

# GET /blocks/height
высота ноды

```
curl http://localhost:6862/blocks/height

curl -X GET "https://clinton.welocal.dev/node-0/blocks/height" -H "accept: application/json" -H "X-API-Key: vostok"
```

# GET /addresses/balance/{address}
узнать баланс ноды