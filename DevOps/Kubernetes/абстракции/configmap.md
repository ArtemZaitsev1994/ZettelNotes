```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-cf-env
data:
  dbhost: postgresql
  DEBUG: "false"
...
```
Содержит reusable настройки для приложений. В поле data определяются все переменные окружения.

Подключается внутри деплоймента 
```
---
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
          envFrom:
          - configMapRef:
            name: my-cfm-env
...
```