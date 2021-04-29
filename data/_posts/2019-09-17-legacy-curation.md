# 기존 curation 분석하면서 공부한 내용 정리

1. A 데이터 기준으로 점수를 계산
2. 1번 기준으로 jaccard similarity를 계산
  - (참고: [https://en.wikipedia.org/wiki/Jaccard_index](https://en.wikipedia.org/wiki/Jaccard_index) jaccard distance)
  - ![jaccard_index.png](/assets/img/cus/data/jaccard_index.png)
  - 간단히 intersection / union 이라고 생각
3. 2번에서 나온 유사도 점수를 기준으로 k-means clustering(군집화, set 만들기)([https://en.wikipedia.org/wiki/K-means_clustering](https://en.wikipedia.org/wiki/K-means_clustering))
4. 3번 군집화 데이터와 B 데이터로 ordering

- issue
- 데이터가 많아지면 많아질수록 jaccard 계산하는 양이 많아 처리 느려짐.
