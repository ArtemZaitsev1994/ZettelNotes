# Выборка самых k элементов
Из этого курса https://www.youtube.com/watch?v=kx-XDoPjoHw&ab_channel=SystemDesignInterview

## Требования
### функциональные требования
 - topK(k, startTime, endTime)

### нефункциональные требования
 - масштабируемость
 - высокая доступность
 - высокая производительность
 - точность

## Компоненты
![[rate limiter.png]]
1. Client Identifier Builder - определяем к какому типу относится реквест, чтобы потом можно было его сматчить с правилами.
2. Rate limiter - матчит request с правилами и смотрит на то, проходим ли мы по этим правилам, или нет.
3.  Throttle Rule Cache - хранит правила, выгруженные из базы.
4.  Throttle Rule Retriever - переодически ходит в базу с правилами и обновляет Throttle Rule Cache.
5.  Request Processor - процессит сам реквест.
6.  Rule Service - [[Db Service]]
7.  Queue - очередь в которую можно сложить запросы для дальнейшей обработки.

## Детальная проработка отдельных частей
Будем использовать Token Bucket Algorithm. Смысл следующий: у нас есть какое-то количество токенов и каждый из реквестов расходует один токен. Эти токены обновляются с течением времени. Обновляем каждый раз, когда проверяем доступ, примерный код ниже.
```
from collections import Counter


class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        return [x[0] for x in Counter(nums).most_common()[:k]]
```

Таким образом мы можем даже корректировать "цену" запроса, передавая параметр tokens.

Этот вариант подойдет для одной машины, если надо распределенную систему, тогда надо думать как шарить данные. 
![[Общение между сервисами.png]]

1. Каждый сервис сообщает каждому, что он поменял данные.я Очень плохо. 
2. Gossip протокол, каждый по цепочке рассказывает о том, что произошло. Мне не нравится.
3. Распределенный кеш где-нибудь в Redis. Больше всего нравится, но здесь желательно что-то придумать с атомарностью, например скрипт на lua.
4. Систему [Master-Slave](Репликация#Master-Slave), но не понимаю этой штуки, если все обращения у нас по идее операции на запись и нет чтения. Или тут речь про шардирование? непонял
5. Тут вообще тему не раскрыли и я нифига не понял(