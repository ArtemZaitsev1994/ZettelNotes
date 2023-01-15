## 1) Публикация с использованием Service
Service - это не прокси, а больше правило в IPTables. Сервис - скорее статический ip, который может быть назначен приложению.

### ClusterIP
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```
Простой сервис, который внутри кластера располагается по DNS записи `metadata.name` и хостит внутри себе по `selector.app`.

### NodePort
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```
Сервис проксирует порты внутри приложения. Открывает все порты на всех нодах кластера. Недефолтный порт делает этот режим неудобным.

### Load Balancer
Работает только в облачных решениях, где есть внутренний контроллер. 
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  type: LoadBalancer
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```

### ExternalName
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ExternalName
  externalName: example.com
```
По сути редирект на какую-то конкретную запись/адрес. Непонятно заачем нужен.

### ExternalIPs
Задаем IP-шник, который цепляют к каждой ноде и по возможности эта нода получает трафик на себя. Ничего не понял.

### Headless
Только для баз данных. Для роутинга трафика записи/чтения. POD-ы будут взаимодействовать не через прослойку services (эффемерный ip), а напрямую друг с другом.

## 2) Ingress

Ingress - манифест, IngressController - приложение внутри кубера.

### IngressController
Абстракция над балансировщиком (ngnix, haproxy, traefic). 
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
  - host: foo.mydomain.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: my-service
            port:
              number: 80
```
annotations - помогает расширить возможности.
В сюда же подключается сертификат tls через секрет.

Желательно, чтобы ингресс-контроллер был запущен на отдельной виртуалке а лучше даже железке.
