Набор pod'ов, который управляет масштабированием этих подов. 
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
  labels:
    app: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - image: nginx:1.12
          name: nginx
          ports:
          - containerPort: 80
...

```

Если реплику поправить напрямую через команду `kubectl edit rs <name rs>` , то мы отредактируем только файл, поды не пересоздадутся. Но новые поды (при падение, например) будут разворачиваться по новой спецификации.

