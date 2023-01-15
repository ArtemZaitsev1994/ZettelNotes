### Limits
Количество ресурсов, которые POD может использовать. Верхняя граница, после которой кубер перезапустит POD.

### Requests
Количество ресурсов, которые резервируются для PODа. Не важно используются ли они.

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-dp
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
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
            limits:
              cpu: 100m
              memory: 100Mi
...
```
**cpu:** 1m - один мили-cpu - одна тысячная процессорного времени одного ядра в системе. Но можно указывать в целых числах количество ядер.
**memory:** 1Mi - 1024

### ResourceQouta
Устанавливает количество доступных ресурсов и объектов для неймспеса в кластере. Это могут быть различные объекты типа: реквесты, лимиты, сервисы, поды.

### LimitRange
Устанавливает ресурсы для объектов кластера внутри неймспейса. Для контейнеров, подов, PVC.
