Абстракция, которая разворачивает поды/реплик-сеты.
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-dp
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

Можно управлять версиями контейнеров с мгновенным переразвертыванием `kubectl set image deployment <name dp> '*=<name-image>'`

### Стратегии передеплоя
- RollingUpdate (по умолчанию) - убивает один (см. maxUnavailable) под в старой версии, поднимает один (см. maxSurge) под в новой и так все поды.
- Recreate - убивает всё сразу и создает всё сразу.

### Настройка RollingUpdate
`spec.strategy.rollingUpdate.`:
- maxSurge - количество реплик которые могут быть созданы поверх сущестсвующих. У нас стоит в rs поддерживать 5 реплик, maxSurge == 2, то есть поверх 5 реплик можем создать 2 новых во время (ре)деплоя.
- maxUnavailable - количество реплик которые могут быть убиты поверх сущестсвующих. У нас стоит в rs поддерживать 5 реплик, maxUnavailable == 2, то есть мы можем сходу убить 2 реплик во время (ре)деплоя.
Оба параметра можно указывать в процентах от общего количества реплик.