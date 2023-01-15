## Job
Создает для себя POD в одноразовом формате. Будет пытаться выполнить задачу до того момента, пока не справится, либо до таймаута.

```
---
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  backoffLimit: 2
  activeDeadlineSeconds: 60
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        args:
        - /bin/sh
        - -c
        - date; echo Hello from k8s
      restartPolicy: Never
...
```
__backoffLimit__ - ограничивает количество попыток выполнить работу.
__activeDeadlineSeconds__ - timeout в секундах
__restartPolicy__ - по умолчанию стоит _Always_. Политика переподнимания контейнеров, если они упадут внутри пода. Если оставить Always, то не увидишь ошибку.

Жобы и поды сами по себе не удаляются. Поэтому внутри spec указываем __ttlSecondsAfterFinished__, например 120. После остановки PODа начнется отсчет.

## CronJob
Создает Job-ы по расписанию. 
```
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      backoffLimit: 2
      activityDeadlineSeconds: 100
      tempalte:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from k8s cron
          restartPolicy: Never
...
```
__concurrencyPolicy__ - энам Allow, Forbid, Replace.
__Successful Job History Limit__ - оставляет количество удачных джобов, остальное чистит. По умолчанию - 3.
__Failed Job History Limit__ - по умолчанию 1.

__startingDeadlineSeconds__ - время от срабатывания крона и до старта работы job-ы, которое мы готовы потерпеть. Если не указан, то после падения CronJob-а, если прошло времени больше чем 100 кроновых промежутков, то CronJob уже не запустится - нужно пересоздавать. А если задан, то измерять прошли ли 100 кроновых промежутков будет внутри значения __startingDeadlineSeconds__, а не прошедшего времени.
Также лучше указывать значение > 10 секунд, иначе можно внутреннюю логику менеджера кронжобов сломать.


