Подписание и проверка подписи
# Для подписи CAdES-BES или sigType = 1
1. Скачать corporate-generator из репозитория: corporate-generator-1.13.0-Dev.jar
2. Конфиг для генерации ключевой пары  (generator.conf):

```json
pki-key-pair-generator {
    crypto-type = gost
    chain-id = V # должен быть как в конфиге сети, к которой будет подключаться нода (https://vault-dev.welocal.dev/ui/vault/secrets/we/show/sato/config файл network.conf строка address-scheme-character = "V")
     key-pair-algorithm = "GOST_EL_2012_256" # или GOST_DH_2012_256
    keystore-password = "vostok"
    out-dir = "./out/" # путь куда будет помещен запрос на сертификат
    cert-request-content {
        CN = "EL_node_0"
        O = "Waves Enterprise"
        OU = "IT Business"
        C = "RU"
        S = "a"
        L = "b"
        extensions {
            key-usage = ["digitalSignature"] # для digital_signature используется алгоритм GOST_EL_2012_256, для key_encipherment и data_encipherment используется – GOST_DH_2012_256
            extended-key-usage = ["serverAuth"]
            subject-alternative-name = "DNS:node-0,IP:0.0.0.0"
        }
    }
}  
```
3. Запустить генератор:

```
java -cp "corporate-generator-1.13.0-Dev.jar:java-csp-5.0.44122-A/*" com.wavesenterprise.generator.pki.PkiKeyPairGenerator generator.conf 
```
В результате будет создан 
- запрос на сертификат вида 3MqFWYnVcBndh3Nc8gz9Ar5LEjnFQFjZX5u.req 
- аккаунт (keystore) в директории `/private/var/opt/cprocsp/keys/<username>/<3MqFWYnV.000>`
- адрес
- публичный ключ

4. Получившийся реквест сконвертировать в base64:
```
base64 -i out/3MqFWYnVcBndh3Nc8gz9Ar5LEjnFQFjZX5u.req
```

5. Подписать реквест https://testgost2012.cryptopro.ru/certsrv/certrqxt.asp и скачать клиентский сертификат .crt

6. Положить клиентский сертификат в keystore:
```
/opt/cprocsp/bin/csptest -keyset -container 3Hk6XCqafgmxePGgXHq1MWBdU4Kc8T1Z55a -keytype signature -impcert user.crt
```
7. Скачать корневой сертификат:
	1. https://testgost2012.cryptopro.ru/certsrv/certcarc.asp или
	2. http://testgost2012.cryptopro.ru/CertEnroll/root2023-2.crt или 
	3. https://testgost2012.cryptopro.ru/certsrv/certcarc.asp)
8. Положить keystore на ноду (https://vault-dev.welocal.dev/ui/vault/secrets/we/list/sato/node-0/wallet/)
- Create secret (имя аналогичное имени в п.3 (3MqFWYnV.000))
- Имя соответствует имени в папке (header.key, masks.key, masks2.key, name.key, primary.key, primary2.key)
- Содержание является результатом преобразования каждого ключа в base64 `("MDYEIGsIB0fGqoTP9...")`
9. Настроить стенд
- Выполнить
```
kubectl edit sts node -n <namespace>
```
- Добавить
	- переменную окружения "ADD_CERTIFICATES" для добавления корневого сертификата в доверенные,
	- папку для сертификатов и
	- связать ее с именем в node-secret.yaml:
```
- env:
  - name: ADD_CERTIFICATES
    value: "true"
...
volumeMounts:
 - mountPath: /node/certificates
   name: node-cert
...
volumes:
- name: node-cert
  secret:
    secretName: node-cert
```
10. Положить корневой сертификат на стенд

- Для этого в файле https://gitlab.web3tech.ru/devops/argocd/ovh/we-argo-v2/-/blob/master/overlays/sato/node-secret.yaml добавить сертификат root2023-2.crt дважды преобразованный base64:
```
apiVersion: v1
kind: Secret
metadata:
  name: node-cert
type: Opaque
data:
  root2023-2.crt: TUlJRkh6Q...
```
11. Чтобы сделанные изменения применить - перезагрузить под у ноды https://argocd.welocal.dev/applications/argocd/sato?view=tree&resource=

12. Подписать данные /pki/sign
```
{
  "inputData": "<base64 input data>",
  "keystoreAlias": "keystore alias",
  "password": "vostok",
  "sigType": 1
}
```
13. Проверить подпись /pki/verify
```
{
  "inputData": "<base64 input data>",
  "signature": "genereted signature",
  "sigType": 1
}
```
# Для подписи CAdES-T или sigType = 3
- К выше указанным шагам добавить настройку в network.conf
```
corporate-features{
    ...
    pki {
        tsp-server = "http://testca2012.cryptopro.ru/tsp/tsp.srf"
    }
}
```
- Скачать корневой сертификат: http://testca2012.cryptopro.ru/cert/rootca.cer
- Положить корневой сертификат на стенд (как в 10 пункте, ссылка)
## Для подписи PKCS7 или sigType = 4
Для проверки подписи необходимо только чтобы корневой сертификат, которым подписывали находился на стенде ссылка.
## Заметки:
- Посмотреть лог на ноде: 
```
kubectl logs -n <namespace> -c node node-0 -f
```
- Подключиться к ноде: 
```
kubectl exec -it -n <namespace> node-0 sh
```
- Корневой сертификат добавить в trust store
```
sudo keytool -import -trustcacerts -alias crypto_cert -noprompt -storepass changeit -keystore <JAVA_HOME>/Contents/Home/lib/security/cacerts -file root2023-2.crt
или
sudo keytool -import -trustcacerts -alias crypto_cert -noprompt -storepass changeit -cacerts -file root2023-2.crt
```
- Корневой сертификат удалить из trust store
```
sudo keytool -delete -trustcacerts -alias crypto_cert -noprompt -keystore <JAVA_HOME>/Contents/Home/lib/security/cacerts
или
sudo keytool -delete -trustcacerts -alias crypto_cert -noprompt -storepass changeit -cacerts
```
- Список корневых сертификатов
```
sudo keytool -list -keystore <JAVA_HOME>/Contents/Home/lib/security/cacerts
или
sudo keytool -list -storepass changeit -cacerts
```
- Кодировать в base64
```
base64 -i file
или
base64 <<< string
```
- Декодировать из base64
```
pbpaste | base64 -d > file
или
base64 -d > fileI > fileOut
```
- Сменить пароль на ключевом контейнере
```
/opt/cprocsp/bin/csptest -passwd -change <new passwd or "" if empty> -passwd <old passwd> -cont <alias>
```