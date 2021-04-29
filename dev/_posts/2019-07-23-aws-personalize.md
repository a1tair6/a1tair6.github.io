# 개인화 추천


as-is
- 컨텐츠 추천 개인화
- 현재 legacy로 jaccard 와 k-means 알고리즘이 사용되고 있음.
- jaccard의 단점인 데이터가 많으면 많을수록 속도가 저하됨.
- 6개월 정도의 사용자 구매와 조회 데이터 사용

to-be 
- 속도를 빠르게
- min-hash 알고리즘 사용(jaccard 대체)
- 추가로 랭킹, 실시간 랭킹 등

- 위 사항으로 개발하려 했으나 AWS Personalize 도입 검토

## AWS Personalize
- url : [https://docs.aws.amazon.com/ko_kr/personalize/latest/dg/how-it-works-dataset-schema.html](https://docs.aws.amazon.com/ko_kr/personalize/latest/dg/how-it-works-dataset-schema.html)
- summit 자료에 상세히 설명 되어 있음. [https://www.slideshare.net/awskorea/ai-amazon-personalize-aws-aws-aws-summit-seoul-2019](https://www.slideshare.net/awskorea/ai-amazon-personalize-aws-aws-aws-summit-seoul-2019)

- 사용자 metadata csv 기준
  - user_id(필수), age, gender
    - 생략
- contents metadata
  - item_id(필수), title, genre
- 사용자 컨텐츠 구매 metadata(raw데이터)
  - user_id, item_id, timestamp
- 사용자 조회 metadata
  - user_id, item_id, item_view_count, timestamp
- batch inference job


### 장점

- 현재 컨텐츠 당 similarity 추천임.(ex. A컨텐츠를 본 사용자들은 모두 같은 dataset을 추천함.)
- personalize는 개별적으로 추천 컨텐츠가 달라짐.(말 그대로 개인추천)
- 매일 배치가 돈다면 매일 추천하는 컨텐츠가 달라질 것으로 예상(확인 중)
- 현재 curation에 신규작품 노출 힘듦(~~personalize 전체 결과 데이터 뽑아서 확인 중~~)
  - 현재 curation 신규 작품 아예 안된다고 볼 수 있음.(구매, 조회 데이터 기반)
    - 컨텐츠 A (12.08)
    - 240만명 중 20770 match
    - 컨텐츠 B (12.08)
    - 240만명 중 10904 match
+ 추후 실시간 랭킹 도입 시 유용 함.


### to-do list
- 국가 별로 따로
- ~~app 개발 create dataset, create solution, create compaigns(describe 등등) + event track~~
- app 일별 스케쥴
- heavy user 추출(처음엔 약 10명 정도로)
- heavy user 들의 현재 curation 화면
- aws personalize 적용 후 화면
- ~~add api~~
- add backoffice (메뉴 필요 X)
- https://{url}/#/personalize

### 질문
- dataset은 3개 이상 추가 할 수 없는지
  - 자문자답 : only 1
- dataset이 바뀌면 solution, campaign을 다시 생성 해야 하는지
  - solution version update
- ~~solution recipes 실패 시 원인 파악~~



### 단점
- dataset 3가지만 가능(group으로 새로 생성하면 됨)
- dataset 바뀌면 새 solution, campaign 생성
- solution 생성하는데 어어엄~청 오래 걸림.(1.9gb 기준 하루 넘어감)

![personalize_alert.png](/assets/img/cus/dev/personalize_alert.png)

### personalize concept architecture
- https://app.diagrams.net/#G1m2EDSbv4Pg7O7PyeIwGTEEBWbiWjf7cY


### 테스트
#### 테스트1(10/17)
- 경과 시간 2시간 이상 소요
- user_meta.csv 26M size, 약 100M row, gender, 나이 있는 사용자만
- user_purchase.csv 45M size, 20190801 이후 구매 데이터
- contents_meta.csv 667K size

#### 테스트2(10/24)
- user_meta.csv, locale 추가, gender, age 없으면 none, 약 1900만 row
- user_purchase.csv 20190101 구매 데이터 + 20190101 사용자 view 데이터 사이즈 26gb 정도라 너무 큼.
- user_purchase.csv 20190501 구매 데이터 + 20190501 사용자 view 데이터 15gb
- contents는 그대로.
- solution recipes 3개중 2개 실패
- 원인 모름
- solution 에러 남
  ![psnlz_solution_err.png](/assets/img/cus/dev/psnlz_solution_err.png)


#### 테스트3(11/28)
- user_meta.csv 기존 locale, gender, age (추후 + membership)
- contents_meta.csv
- 장르 통합으로 새로 추가
- user_purchase.csv (1.9GB)
- solution 생성되는데 하루 넘김.


#### 테스트4(12/19)
- 신규로 나온 batch inference jobs 테스트
- 전체 1.6gb 생성
- 시간 오래 소요 될 것으로 생각 됨.
- User-meta를 줄이거나 solution num을 줄이거나
- 1만건 2min 소요
- 2472510 건 40,000sec 소요

#### Solution version metrics
- 아래 항목의 수치가 괜찮다고 AWS 담당자에게 답변을 받았음.
- Normalized discounted cumulative gain
- Precision
- Mean reciprocal rank
- Coverage Info

## 결론
- **But, 나만 만족하는 결과 였음.** 
- 구매 or 조회가 없었던 새로운 사용자들 같은 경우 내부적으로 cold start 를 내보내어 보통 같은 작품을 추천함.
- 이후 진행 안함.