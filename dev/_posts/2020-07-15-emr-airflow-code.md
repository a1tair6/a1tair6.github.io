# airflow emr step template

- airflow scheduler 로 emr 띄웠다 내렸다
- public cloud(gcp, aws etc.) 는 클릭 몇 번으로 hadoop cluster 를 띄울 수 있음.
  (기본 개념이 너넨 인프라 신경 쓰지마 우리가 해줄께, 이부분은 다시..)

#### 참고
- [https://github.com/apache/airflow/blob/master/airflow/providers/amazon/aws/example_dags/example_emr_job_flow_manual_steps.py](https://github.com/apache/airflow/blob/master/airflow/providers/amazon/aws/example_dags/example_emr_job_flow_manual_steps.py)