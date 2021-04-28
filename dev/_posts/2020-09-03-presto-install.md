# presto 설치 with helm
- configmap 자동으로 만들어서 적용 버전
- variables hostname, ip 입력 받아서 적용

- docker 에서 s3에 있는 thanos-udf-1.0.jar 가져옴.

```
step 1
thanos-udf build & deploy (gitlab-ci 적용 해야 함)

step 2
git clone {git url}
./build-remote.sh 334
  
tag 변경 후 사내 docker에 upload
  
step 3  
각 yaml 파일 수정 후, Chart.yaml version up
index 생성 후 git push
$ helm package presto
$ helm repo index stable --url {git url}
 
  
step 4
rancher Catalogs 에서 refresh 후 app에서 variable 입력 후 launch
variable
* hdfs.config.hostname={hostname}
* hdfs.config.ip.nn1={ip}
* hdfs.config.ip.nn2={ip}
```