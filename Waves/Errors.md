Если при запуске интегро тестов получаем `failed to solve: lsetxattr com.apple.quarantine /ASN1P.jar: operation not supported` то проблема в докере, возможно надо другую версию взять

Если при запуске интегро тестов OutOfMemoryError, то надо запускать так
```
sbt -mem 4096 integrationTests/Test ...
```