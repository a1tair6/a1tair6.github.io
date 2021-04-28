# hue 4.5.0 설치 with helm chart

- helm chart hue 사용
- rancher catalogs에 hue helm 주소 추가
  - [https://helm.gethue.com](https://helm.gethue.com)
- config만 수정 해서 사용

- **issue**
hue 4.7.0 버전 ui 상에서 분 셀렉트박스가 안나오는 버그가 있음.
dockerhub에서 4.6.0의 가장 최근 버전인 gethue/hue:20200410-135001 찾아서 사용