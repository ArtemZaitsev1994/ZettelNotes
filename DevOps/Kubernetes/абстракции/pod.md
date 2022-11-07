Минимальная абстракция, может содержать в себе несколько контейнеров. Минимальная единица деплоя внутри кубера.

```
---
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-1
spec:
  containers:
    - image: nginx:1.12
      name: nginx
      ports:
      - containerPort: 80
...

```

В describe можно посмотреть QoS Class, который говорит о том что там с выделенными [[resources|ресурсами]].