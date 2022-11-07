```
create table sales(
salesid interger not null encode mostly16,
sallerid integer not null,
listid integer not null,
primary key(salesid),
foreign key(sellerid) references users(userid),
distkey(listid),
compound sortkey(listid, sellerid)
);
```

* Data type - тип данных
* Constraints - ограничения (not null, null, primary key, foreign key)
* Encoding - сжатие/компрессия данных
* [[Sort key]] - как храним данные, организация партиций
* [[Distribution style]] - как раскладываем данные среди кластеров

