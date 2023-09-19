# POST `/pki/verify`
Ошибки:
- `Authorization error. Missing authorization` metadata говорит о том что нужно авторизоваться
# POST `transactions/signAndBroadcast`
## Создать контракт (103)
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
```scala
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
# GET /contracts/{contractId} 
получить состояние контракта

# GET /contracts/info/{contractId}
получить информацию о контракте

# GET /confidential-contracts/tx/{executable-tx-id}
получить состояние КСК

# POST /confidential-contracts/call
выполнить запрос к КСК
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