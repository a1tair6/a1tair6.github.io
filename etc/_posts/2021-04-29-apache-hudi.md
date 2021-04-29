# Apache hudi 살펴만 보기

- 기존 hive, spark 데이터 처리 할 때 증분 데이터 처리가 제일 핵심이였음.
- emr 관련 된 내용 찾다가 apache hudi 관련 된 내용을 발견.
- 가장 중요한 핵심은 증분 데이터 처리, Upsert, CDC
- Spark과 Flink에서 사용 가능 함.
- 아직 Stable version은 안나옴. 현재 버전 0.8.0

## hudi 작동 방식
- CoW(Copy on Write) : Parquet 형식 사용
- MoR(Merge on Read) : Parquet, Avro 형식 사용
- HIVE_SYNC_ENABLED_OPT_KEY 속성으로 hive metastore 에 저장
- hive metastore 에 read-optimized view와 real-time view(postfix _rt) 생성


## hudi 사용 방법
- 도입 하면서의 이슈는 다음에
- [https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hudi-work-with-dataset.html](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hudi-work-with-dataset.html)

## 참고
- [https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hudi.html](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hudi.html)
- [https://hudi.apache.org/](https://hudi.apache.org/)