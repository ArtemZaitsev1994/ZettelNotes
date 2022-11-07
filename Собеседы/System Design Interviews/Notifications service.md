# Распределенная очеред сообщений
Из этого курса https://www.youtube.com/watch?v=bBTPZ9NdSk&ab_channel=SystemDesignInterview

## Требования
### функциональные требования
 - createTopic(topicName)
 - publish(topicName, message)
 - subscribe(topicName, endpoint)

### нефункциональные требования
 - масштабируемость
 - высокая доступность
 - высокая производительность
 - надежное

## Компоненты
![[notifications service.png]]
1. [[Load Balancer]]
2. [[FrontEnd Service]] связан с Metadata Service. Зачем? Ответ автора видео: 
```
	Here are some reasons why we might want to call Metadata service from FrontEnd:
	- We may implement a smarter partitioning logic for messages. E.g. let's say we choose Kafka as a Temporary Storage. Without knowing anything about subscribers, we just blindly send a message to a single topic in Kafka. With information about subscriptions in our hands, we may send messages for different subscription types to different topics in Kafka. E.g. messages that need email delivery go to one Kafka topic, messages to send to HTTP endpoints go to another topic and so on.
	- If our Notification service API allows a more sophisticated message validation/filtering logic, we may also store these attributes in Metadata service and do some of these activities in the FrontEnd service. There are pros and cons for this as well, as we do not want to overload FrontEnd with "too much" logic. 
	- From operability standpoint it also might be preferable that only one component (FrontEnd) talks to Metadata service for all the operations (createTopic, subscribe, publish, etc.).
```
3.  Metadata Service - имплементация [фасада](Ambassador.md) перед доступом в базу. Предполагается что будет кешировать запросы и может быть зашардирован. [[Шардирование]]
4.  Temporary Storage - база данных, которая будет хранить оповещения короткое время, пока мы их не отправим пользователю. База должна быть scalable, partition tolerant и high available. Здесь не нужны транзакции, связи и хранение данных для BI, WH, statistics. Выбор: Cassandra, Amazon DynamoDB
5.  Sender - в разных потоках шерстит temporary storage на наличие новых оповещений. Ходит в MS для получения актуальной информации "кому слать". Шлет сообщения.
![[sender notifications.png]]

