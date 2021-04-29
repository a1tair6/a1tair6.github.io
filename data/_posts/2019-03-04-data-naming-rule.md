# naming rule

- 체계적인 데이터 관리
- 데이터의 가용성, 통합성을 위한 정책으로 데이터 품질을 위한 naming rule 을 정함.
- data governance 의 일환

## 주제영역


|영역 구분|주제영역 1 level|주제영역 2 level|database 명|table prefix|description|
|---|---|---|---|---|---|
|통합 영역(DW)|공통관리(Common Management, CM)|공통코드관리(Common Code Management, C)|D_CO|dco||
|||조직관리(Organization Management, O)||||
|||지역관리(Area Management, A)||||
||컨텐츠관리(Contents Management, CT)|컨텐츠관리(Contents Management, CT)|D_CT|dct||
|||이벤트관리(Event Management, EV)||||
||결제관리(Payment Management, PM)|결제 관리(Payment Management, P)|D_PM|dpm||
|||구매 관리(Buy Management, B)||||
|||쿠폰 관리(Redeem Management, R)||||
||구성관리(Configuration Management, CF)|구성관리(Configuration Management, C)|D_CF|dcf||
|||품질관리(Quality Management, L)|D_CF|dcf||
||서비스관리(Service Management, SM) / 고객관리(Customer Management, CM)|고객체감품질관리(Customer Feel Quality Management, F)|D_CM|dcm||
|||고객관리(Customer Management, C)||||
|||구독관리(Subscription Management, S)||||
|||파트너관리(Partner Management, P)||||
||성능 관리(Quality Management, QM)|오류관리(Error Management, E)|D_QM|dqm||
|||통계관리(Statics Management, S)||||
|분석 영역(DM)|컨텐츠분석(Content Analysis, CT)|컨텐츠분석(Content Analysis, CT)|M_CT|mct||
||결제분석(Payment Analysis, PM)|결제 관리(Payment Management, P)|M_PM|mpm||
||서비스분석(Service Analysis, SA) / 고객분석(Customer Analysis, CM)|체감품질분석(Feel Quality Analysis, F)|M_CM|mcm||
|||경험품질분석(Experience Quality Analysis, E)||||
|||고객분석(Customer Analysis, C)||||
|||파트너분석(Partner Analysis, P)||||
||성능분석(Quality Analysis, PA)|오류분석(Error Analysis, E)|M_QA|mqa||
|||통계분석(Statics Analysis, S)||||


## Table naming rule
- Prefix는 주제영역(Database) 에서 테이블Prefix.
- `{prefix}_{table 성격에 따른 Name}_{version}_{master|History}_{Period}`
- `{prefix}_{table 성격에 따른 Name}_{master|history}_{sql|ds|nosql}_{Partition Period}`
- ex) DW 영역의 1일 파티션 상품 클릭 히스토리 테이블 → d_cm.dcm_product_user_click_count_hst_1d

## storage naming rule
### ODS(Operation Data Store)
- s3://{company}/{database_name}.db/{table_name}/partition column
- spark 에서 equal 구분자로 다른 spec이 있음(경로에 못 씀, 더 확인하고 싶지만 시간 없어서 pass)
  - db={database} → {database}.db 로 변경
  - tb도 삭제
- 현재 데이터도 별로 없고, 의미 없으므로 삭제(추후에 데이터 많아지면 도입)
  - version 경로
  - period 경로

### DW(Data Warehouse)
- Path
  - hdfs://{cluster name}/
- Coordinator
  - dw_{데이터의 성격}_{period}
  - prefix는 dw_
  - suffix는 ETL하는 period 주기
    - ex) dw_event_data_1d
- Workflow
  - ODS에서 DW로 ETL하는 Workflow Job
  - dw_{tablename}
  - prefix는 dw_
  - 테이블명은 표준화를 거쳐 생성된 Name이기 때문에 주기 표시됨.


### MART
- Coordinator
  - mt_{데이터의 성격}_{period}
  - prefix는 mt_
  - suffix는 ETL하는 period 주기
    - ex) mt_event_data_1d
- Workflow
  - DW에서 MART로 ETL하는  Workflow Job
  - mt_{tablename}
  - prefix는 mt_
  - 테이블명은 표준화 거진 DW 테이블 명