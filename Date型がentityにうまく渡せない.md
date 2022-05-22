```
task.setDeadline(((Timestamp) result.get("deadline")).toLocalDateTime());
```
* 上記のように、データベースから取得した値は、TimeStamp形で渡ってくるので、map型から取り出したときは、いったんTimeStampにキャストする必要がある。
* そのあとに、entityの型（例であればLocalDateTime）に直して、setする必要があります。