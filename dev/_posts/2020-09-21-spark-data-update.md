# spark jdbc savemode 종류

- spark sql jdbc는 4가지 제공
  - Overwrite
    - truncate option 이 있음.
  - Append
  - ErrorIfExists
  - Ignore

- spark-sql lib - JdbcRelationProvider 코드
- [spark_savemode.png](/assets/img/cus/dev/spark_savemode.png)


- ON DUPLICATE KEY UPDATE 쿼리 해야 할 일이 생김.

- ```sql
  INSERT INTO table (columns)
  values(values)
  ON DUPLICATE KEY UPDATE
  ```
  

- JdbcUtils.scala 를 custom 해서 사용 함.
  ```scala
  val sql = s"INSERT INTO $table ($columns) VALUES ($placeholders) ON DUPLICATE KEY UPDATE $duColumns"
    conn.prepareStatement(sql)
  ```
  
- **주의. spark에서 지원 안 하는데는 이유가 있을테니 하지 말 것**