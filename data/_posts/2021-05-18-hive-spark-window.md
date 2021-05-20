# hive, spark window

- 기존 rdb와 비슷
  - rows : 현재 행 기준 몇개의 행을 포함하는지
  - range : 현재 행 기준 어떤 값의 범위를 포함하는지
  - between A and B
  - unbounded preceding : 시작 지점
  - current row : 현재 row


## hive

```hiveql
rows between unbounded preceding and current row
```
    
## spark

```sparksql
rowsBetween(Window.unboundedPreceding, Window.currentRow)
```