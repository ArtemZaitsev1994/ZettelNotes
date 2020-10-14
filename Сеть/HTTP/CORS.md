Cross-Origin Resource Sharing - механизм, который использует дополнительные HTTP-заголовки, которые определяют поведение получения ресурсов с других доменов. Если сайт запрашивает ресурсы с других источников, отличных от домена (домен, протокол, порт), на котором он сейчас находится, то браузер действует исходя из CORS. [ссылка на mozila](https://developer.mozilla.org/ru/docs/Web/HTTP/CORS)

[XMLHttpRequest](https://developer.mozilla.org/ru/docs/Web/API/XMLHttpRequest) и [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) по умолчанию следуют политике одного источника.

CORS описывает набор источников, которые могут брать информацию. В отличии Content-Security-Policy от CORS выполняется на запрашиваемой стороне и фактически отвечает тем, имеет ли он право общаться (как сервер) с клиентом, который что-то запрашивает.

#### Заголовки ответа

* **Access-Controll-Allow-Origin** - устанавливает список ресурсов которые имеют доступ. Например:
	`Access-Control-Allow-Origin: http://mozilla.org`.
* **Access-Control-Expose-Headers** - создает белый список заголовков, к которым у браузера будет доступ. Например:
	`Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header`
* **Access-Control-Max-Age** - устанавливает время в секундах на сколько браузер может кешировать ответ презапроса. Например:
	`Access-Control-Max-Age: <delta-seconds>`
* **Access-Control-Allow-Credentials** - устанавливает флаг, может ли запрос быть выполнен с использованием учетных данных. Пример:
	`Access-Control-Allow-Credentials: true`
* **Access-Control-Allow-Method** - в заголовке указывается список методов, с которыми можно обращаться к серверу. Пример:
	`Access-Control-Allow-Methods: <method>[, <method>]*`
* **Access-Control-Allow-Headers** - в заголовке указывается список заголовков, с которыми можно обращаться к серверу. Пример:
	`Access-Control-Allow-Headers: <field-name>[, <field-name>]*`
	
#### Заголовки запроса
* **Origin** - URI откуда будет производиться загрузка. Синтаксис:
	```
	Origin: ""
	Origin: <протокол> "://" <имя_хоста> [ ":" <порт> ]
	```
* **Access-Control-Request-Method** - указывает какой метод собирается использовать сервис для доступа. Синтаксис:
	`Access-Control-Request-Method: <method>`
* **Access-Control-Request-Headers** - указывает какие заголовки собирается использовать сервис для доступа. Синтаксис:
	`Access-Control-Request-Headers: <field-name>[, <field-name>]*`