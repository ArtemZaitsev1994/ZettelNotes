Хранение приватной информации.
```
---
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
stringData:
  dbhost: postgresql
  DEBUG: "false"
...
```

создать секрет `kubectl create secret generic test --from-literal=test1=asdf --from-literal=dbpassword=qwe123`

Можно доставать [[envs]] из секретов
```
env:
- name: test
  valueFrom:
    name: secrets-test
    key: foo
```
