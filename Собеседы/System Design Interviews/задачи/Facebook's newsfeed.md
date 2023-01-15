Designing Facebook’s Newsfeed [отсюда](grok_system_design_interview).

## Требования
### функциональные требования
 - Формирование ленты новостей из постов людей, групп и страниц на которые подписан пользователь.
 - Пользователь может быть подписан на множество людей и групп.
 - Новости могут содержать картинки, видео и текст.
 - Возможность динамически формировать страницу когда приходят новые посты.
 
### нефункциональные требования
 - Система должнауметь формировать ленту новостей "на лету" - максимальная задержка 2секунды.
 - Сообщение должно попадать в ленту пользователя не дольше чем через 5 секунд.

## Оценка данных и определение ограничений
Предположим что пользователь имеет в среднем:
 - 300 друзей
 - 200 подписок
 - 300М активных пользователей в день
 - 5 раз в день запрашивают ленту новостей
 - в каждой новостной ленте должно быть до 500 постов
 - пост весит в среднем 1KB

Запросов в день: `300М * 5 = 1.5B`

Запросов в секунду: `1.5B / (30 * 24 * 3600) = 17 500 req/s`

*__Хранилище:__*
Память на новостную ленту одного пользователя: `500 * 1 = 500КB`

Общий объем данных: `300М * 500KB = 150TB`

*__Трафик:__*
Исходящий: `17 500 * 500 bytes = ~9 MB/s`

*_Summary:_*
 - Запросов в секунду - 17500
 - Исходящий трафик 9MB/s
 - 150TB для хранения новостей на данный момент

## Проектируем API
 - getUserFeed(api_dev_key, user_id, since_id=None, count=None, max_id=None, exclude_replies=None)
	Parameters: 
	 * api_dev_key (string): The API developer key of a registered can be used to, among other things, throttle users based on their allocated quota.
	 * user_id (number): The ID of the user for whom the system will generate the newsfeed.
	 * since_id (number): Optional; returns results with an ID higher than (that is, more recent than) the specified ID.
	 * count (number): Optional; specifies the number of feed items to try and retrieve up to a maximum of 200 per distinct request.
	 * max_id (number): Optional; returns results with an ID less than (that is, older than) or equal to the specified ID.
	 * exclude_replies(boolean): Optional; this parameter will prevent replies from appearing in the returned timeline.
	 Returns: (JSON) Returns a JSON object containing a list of feed items.

## Выбор базы данных
Так как нам надо:
 - Хранить миллиарды записей.
 - Есть реляционные связи.
То мы выбираем SQL решение. Одно из самых надежных и популярных - PostgreSQL. 

Таблицы
__Entity__ - группы/страницы
PK:
	Name: varchar(20) - до 20 bytes
Type: tinyint - 1 byte
Description: varchar(512) - до 512 bytes
CreationDate: datetime - 8 bytes
Category: smallint - 2 bytes
Phone: varchar(12) - до 12 bytes
Email: varchar(20) - до 20 bytes

__User__
PK:
	UserID: int- 4 bytes
Name: varchar(20) - до 20 bytes
Email: varchar(32) - до 32 bytes
CreationDate: datetime - 8 bytes
LastLogin: datatime - 8 bytes

__UserFollow__
PK:
	UserID: int - 4 bytes
	EntityOrFriendID: int - 4 bytes
Type: tinyint - 1 byte

__FeedItem__
PK:
	FeedItemID: int - 4 bytes
Contents: varchar(256) - до 256 bytes
EntityID:: int - 4 bytes
LocationLatitude: int - 4 bytes
LocationLongitude: int - 4 bytes
CreationDate: datetime - 8 bytes
NumLikes: int - 4 bytes

__FeedMedia__
PK:
	FeedItemID: int - 4 bytes
	MediaID: int - 4 bytes
	
__Media__
PK:
	MediaID: int - 4 bytes
Type: smallint - 2 bytes
Description: varchar(256) - до 256 bytes
Path: varchar(256) - до 256 bytes
LocationLatitude: int - 4 bytes
LocationLongitude: int - 4 bytes
CreationDate: datetime - 8 bytes

## Общий вид схемы
![[Facebook newsfeed.png]]

## Детальная проработка отдельных частей
### Feed generation
Я не совсем понял ограничения и требования, но мое решение, базируясь на решении из статьи, такое:
Новости генерируются в следующей последовательности действий:
1. Достаем все посты пользователей и группы, к которым имеет отношение пользователь. Примерный запрос.
```
(
SELECT FeedItemID FROM FeedItem WHERE UserID in (
	SELECT EntityOrFriendID FROM UserFollow WHERE UserID = and type = 0(user))
) UNION (
SELECT FeedItemID FROM FeedItem WHERE EntityID in (
	SELECT EntityOrFriendID FROM UserFollow WHERE UserID = and type = 1(entity))
) ORDER BY CreationDate DESC LIMIT 100
```
2. Ранжируем посты и собираем их в кеш. Кеш - хэшмап с ключем айдишник пользователя, а значение - эта самая лента новостей, записанная в структуру. 
```
Struct {
	LinkedHashMap feedItems;
	DateTime lastGenerated;
}
```

Новые посты приходят через очередь сообщений и мы их впихиваем в наш связный список. Если нет списка, то генерируем. 
\*\*Тут может быть момент, что если мы имеем сгенеренную ленту на 100 новостей и впихнем туда новую, то не факт, что эта лента совпадет с той, которую бы мы генерировали с нули. Здесь либо пишем такой алгоритм, чтоб все соблюдало инварианты, либо забиваем на это.

### Оповещени пользователя
Есть несколько вариантов сообщить пользователю о том, что появились новости - websockets, longpolling, server push.
LongPolling нам хватит за глаза, если у нас нет уже вебсокетного соединения с каким-нибудь гейтвеем.

Из вариантов: push и pull.
Мы будем использовать гибридный метод оповещений. То есть, если пришел новый пост и о нем надо сообщить огромному количеству пользователей, то мы просто молчим. Тут либо клиенты ставят таймер и сами делают пулл по шедулеру, либо просто по новому запросу. А для малого количества пользователей сами шлем оповещения.

## Data Partitioning and Replication
Конечно прибегаем к [[Шардирование]] и [[Репликация]]. Аппликешн надо шардировать по пользователям, чтобы можно было бы хранить кеш. Базу шардировать не получится, разве что по времени. Все старые новости идут в старую базу.
Возможно, как вариант, в базу со слишком старыми новостями сразу записывать новостные ленты, сгенерированные. Или мы храним только 500 как сказали выше?

## Cache
В кеше храним только новостные ленты для 20 процентов активных пользователей LRU-cache.

## Балансеры
Балансеры ставим перед аппликешн сервером, перед сервером базы данных и сервером кеша.