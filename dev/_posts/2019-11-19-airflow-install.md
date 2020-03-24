# apache airflow 설치 with rancher, helm

- kubernetes helm 사용
- oozie, luige 와 비슷한 scheduler
- python으로 쉽게 개발 가능
- data pipeline 재설계 전 기존 ushd-flow cluster에 있는 그대로 옮기기 위해

## airflow 설치
- rancher dev-lz-edw cluster 선택 -> dev-lz-edw-utli projects 선택 -> Apps 클릭 -> Launch 클릭 -> airflow View Defailts 클릭

## helm 설정 변경
- Configuration Option에서 Answers 추가
```
dags.git.url {git url}
dags.initContainer.enabled true
```


## ConfigMaps 설정 변경
```
AIRFLOW__CORE__DONOT_PICKLE true
```

## redeploy
- airflow-flower, airflow-scheduler, airflow-web, airflow-worker 