HTTP-3 - это в большей мере попытка внудрить новый транспортный протокол, не ломая совместимость и с возможностью запускать на всех устройствах. Боль тут в апгрейде всех сетевых устройств, которые раскиданы по всему миру: фаерволы, маршрутизаторы и тд. У каждого из таких устройств может быть своя реализация TCP и поэтоу внедрять какой-либо новый TCP2 не предоставляется возможным. Для этого новый протокол QUIC было решено писать поверх UDP, который по факту известен каждому древнему дерьму.

Хоть QUIC написан поверх UDP, он все равно поддерживать все фишки TCP. Он устанавливает соединение, подтверждает получение/доставку и тд.

Основные особенности:
 - QUIC также завязан на TLS и имееть глубокую интеграцию, что делает его всегда зашифрованным, установку соединения быстрее и поддерживать реализацию QUIC возможно только на оконечных машинах (из-за шифрования).
 - Мультиплекцсирование запросов. QUIC знает о том, что грузит разные файлы и создает для каждого файла разные потоки внутри одного соединения. когда TCP на это не способоен и блочит весь поток, если ему нужно сделать ретрай.
 - QUIC использует CID каждого соединения, чтобы при сбоях/разрыве можно было не пересоздавать соединение, продолжить его использовать определяя себя по CID.

Минусы:
 - QUIC не будут внедрять все компании подряд, так как он мешает мониторить трафик; для оконечного пользователя квик безопаснее, но для ряда компаний делает невозможным предоставлять свои услуги, например фаерволы и DPI
 - Шифрует каждый пакет отдельно, что тратит больше ресурсов, чем тот же TLS over TCP

https://habr.com/ru/company/southbridge/blog/575464/