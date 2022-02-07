# k8s pod에 EBS volume 마운트 이슈

- airflow kubernetesPodExecutor 사용 중
- 여러 pod에서 같은 volume 마운트가 실패

## 원인
- k8s에서 EBS는 ReadWriteOnce만을 지원

## 해결
- affinity 로 node를 지정해서 pod 실행