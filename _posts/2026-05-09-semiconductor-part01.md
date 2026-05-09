---
category: "Semiconductor Sentiment"
---
## 실시간 경제 뉴스 감성 분석 파이프라인 구축기 (Part 1)


### 1. 프로젝트 시작 계기 (Motivation)
저번까지 다양한 데이터를 다루고 파이프라인을 구축해 왔지만, "데이터를 적재하는 것을 넘어, 그 위에서 돌아가는 AI 모델의 라이프사이클을 직접 통제해 보고 싶다"는 목표가 생겼다.
ML Flow 에 대해서 궁금하기도 했고. 난 데이터랑 AI 둘 다 좋아해서 ㅎㅎ
그리고 ai 때문에 반도체 뉴스가 많이 올라올 거 같아서 분야는 반도체 산업으로 선택했다.

그래서 완벽하게 정제된 토이 데이터셋(Toy Dataset) 대신, 끊임없이 변하는 날것의 데이터(Real-world Data)를 다뤄보기 위해 '글로벌 반도체 뉴스 감성 분석(Sentiment Analysis)' 프로젝트를 시작했습니다. 뉴스의 텍스트 데이터를 수집하고, AI 모델로 긍정/부정을 평가한 뒤, 이 모델들의 성능을 체계적으로 추적(ML Tracking)하는 것이 이번 프로젝트의 핵심이다.

### 2. 전체 아키텍처 및 파이프라인 (Architecture)
본 프로젝트는 확장성과 효율성을 고려하여 클라우드 및 최신 데이터 스택을 적극 활용하려고 했다. 
전체 구조는 아래와 같다. 

Data Source: NewsAPI (글로벌 반도체, NVIDIA, TSMC 등 영문 기사 수집)

Storage (Bronze Layer): AWS S3 (Raw 데이터를 Parquet 포맷으로 일자별 파티셔닝 적재)

Processing: Databricks (PySpark / Pandas)

AI Models: Hugging Face 사전 학습 모델 (Baseline 모델 vs. FinBERT)

MLOps Tracking: MLflow (실험 파라미터 및 메트릭 로깅)

### 3. 마주한 문제와 트러블슈팅 (Troubleshooting)
오늘은 간단하게 환경 세팅과, Databricks 에서 두 개의 모델을 실험 해봤다. 역시 예상대로의 결과이지만, 그래도 재밌었다. 이게 목표가 아니니까!

#### Issue: Databricks Serverless 환경의 S3 접근 차단
상황: Databricks 노트북에서 Spark 코어 설정(spark.conf.set)을 통해 S3 자격 증명을 주입하려 했으나, 최신 Serverless Compute의 강력한 보안 정책(Unity Catalog)으로 인해 접근이 원천 차단됐었다. 
Free Tier 를 사용하고 있어서 (Free Tier인게 이유였다.) 흠 어떡하지.. 고민하다가 우회를 하기로 했다.

해결 (우회 전략): 클러스터 생성 권한이 제한된 상황에서, Spark 엔진 대신 Pandas(pd.read_parquet)를 앞단에 내세워 메모리로 데이터를 먼저 읽어오는 방식을 택했다. Python 환경 변수에 임시로 자격 증명을 주입하여 S3를 뚫고 데이터를 가져온 뒤, 이를 다시 분산 처리용 Spark DataFrame으로 변환(spark.createDataFrame)하여 문제를 우회 해결했다.

### 4. 배운 점
- 포맷의 중요성: JSON보다 Parquet

전에 Parquet 이 얼마나 효율적인지 경험해서, 이번 프로젝트는 바로 Parquet 가지고 시작했다.
데이터 수집 단계부터 파싱 리소스가 큰 JSON 대신 Parquet 포맷을 채택했고, 덕분에 이후 Databricks로 데이터를 읽어 들일 때 편했다.

- 도메인 특화 AI 모델의 위력과 MLflow의 존재 이유

동일한 뉴스 데이터를 가지고 두 가지 모델을 비교 실험했습니다.

Baseline 모델 (SST-2 기반): "삼성전자 노동 쟁의(labor dispute)" 뉴스를 일반적인 문맥으로 해석해 '긍정(Positive) 99%'로 잘못 분류했다. (당연한 결과였다)

FinBERT (금융 특화 모델): 경제적 관점에서 생산 차질 리스크를 정확히 이해하고 '부정(Negative) 95%'로 정확하게 분류함. (역시 역시! labor dispute 이라는 걸 잘 잡았다)

단순히 "FinBERT가 더 좋네"가 아니라, MLflow를 통해 각 실험의 모델 이름, 샘플 사이즈, 긍정 뉴스 비율을 대시보드에 나란히 로깅함으로써 "왜 이 모델을 프로덕션에 배포해야 하는지"를 알게 되었다.



------

(Part 2에서 계속... 다음 편에서는 최고의 성능을 보여준 FinBERT 모델을 Model Registry에 등록하고 배포 파이프라인을 구축하는 과정을 다룹니다!)
