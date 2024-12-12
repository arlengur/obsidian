Просматриваем:
- qoq-ao-test.qiwi.com - тест логи
- qoq-ao.qiwi.com - продакшн логи

Выбрать дашборд: ao-ads-fraud-errors-overview-dashboard

 Prod errors:
 - Таймауты листов и аггрегатов, репортим, и уточняем они только ночью или в разное время

Test errors:
- Can't get value with TTL for (ListEntity(10:117:qd_xml_agt_sum,None):5035662-2024-9-19) - и ошибка вида: No table in cassandra found - можно добавить таблицу в кассандру: https://ao-ads-toolbox.testing.qiwi.com/lists/lists?list=&key=
- Timeout exception occurred - и ошибка на сурке вида: Exception when sending request: POST http://antifraud-gateway-test.qiwi.com:80 - можно не репортить, гейтвей - точка входа, если ДМ или ирис таймаутят, то и гейтвей не успеет ответить сурку
- Таймаутятся листы если
	- на дм: rule exception 
	- на дм: ListReader error statusCode=500, ex=Left({"key":"78016436856","version":null,"operation":"check","message":"","list":"10:common:blacklist_toaccount"})
	- на лист ридере: Can't get value for (ListEntity(10:common:blacklist_toaccount,None):78016436856) из-за DriverTimeoutException
- Игнорим ошибки вида
	- Auth failed, LOGIN_FAILED
- Таймауты аггрегатов и/или листов, то про таймауты ДМ можно не писать
	- DM request error - ao-ads-antifraud-gateway
	- Internal server error, sending 500 response - ao-ads-decision-maker  
	- Http code 500 Internal Server Error DM response - ao-ads-antifraud-gateway


```
	

```