---
category: "Semiconductor Sentiment"
---

## 실시간 경제 뉴스 분석 Part 2


### 1. Part 2의 목표: Production 을 해보자!
저번에 S3와 Databricks를 연결해 데이터 파이프라인을 구축하고, 
MLflow를 통해 두 가지의 AI 모델(Baseline vs. FinBERT)의 성능을 비교해 보았다. 
그 결과, 경제 뉴스의 문맥을 정확히 파악하는 FinBERT를 사용하기로 했다. (당연한 결과지만 실험해보고 싶었다)

오늘은 이 모델을 Production 으로 만들고, 백엔드 서버에서 언제든 끌어다 쓸 수 있는 구조 Model Registry 를 하는 것이였다.


### 2. 마주한 문제들과 트러블슈팅
모델을 레지스트리에 등록하는 과정에서 몇 가지 에러를 만났지만 모두 어려운 에러는 아니였고, Databricks 의 보안 때문에 그랬다. (저번과 마찬가지로 ㅎㅎ)
뭐, 그러면서 MLOps 의 최신 표준에 대해서 생각해보는 시간을 가지기도 했다.


#### Issue 1: 사소한 의존성 누락 (ModuleNotFoundError: No module named 'torchvision')
- 모델 패키징 중 torchvision 모듈이 없다는 에러 발생했다. 

- Hugging Face의 transformers 라이브러리가 내부적으로 PyTorch 생태계의 비전 라이브러리 의존성을 체크하는 과정에서 발생한 환경 문제였다.

- Databricks 클러스터 환경 세팅(%pip install)에 torchvision을 명시적으로 추가하여 해결했다.
- 첫 셀에 %pip install transformers mlflow torch torchvision 라고 적어주면된다.


#### Issue 2: 서명 없는 계약서 거부 (Model did not contain any signature metadata)
- 모델 파일(Artifact)을 MLflow에 업로드하려는 순간 발생한 에러.

- Databricks의 최신 보안 거버넌스인 Unity Catalog가 작동했기 때문이었다. 엔터프라이즈 환경에서는 백엔드 개발자의 혼란을 막기 위해, 이 모델이 어떤 형태의 Input를 받고 어떤 Output를 주는지 Signature를 반드시 제출하라고 강제한다.

- mlflow.models.signature의 infer_signature 함수를 도입하면 된다. 모델에 샘플 텍스트를 통과시켜 입출력 형태를 자동으로 분석하게 한 뒤, 그 명세서를 모델과 함께 등록하여 거버넌스 규칙을 통과했다.



#### Issue 3: Stage의 종말, 그리고 Alias의 등장 (Method 'transition_model_version_stage' is unsupported)
- 등록된 모델을 기존 방식인 'Production Stage'로 승격(Transition)시키려다 마주쳤다.

- 과거 MLflow는 모델 상태를 4가지(None, Staging, Production, Archived)로만 고정해 뒀다. 하지만 실무 배포 환경이 복잡해지면서, Unity Catalog는 Stage 방식을 전면 폐지하고, 태그처럼 이름을 붙일 수 있는 Alias 시스템을 최신 표준으로 채택했다.

- 코드를 최신 표준으로 마이그레이션했다.
- transition_model_version_stage 대신 set_registered_model_alias를 사용하여 모델 버전에 @production 이라는 유연한 별칭(Alias)을 부여하는 데 성공했다.


------


이제 Alias 배포를 구축해 둔 덕분에, 이제 백엔드에서 서비스 서버(FastAPI, Spring 등)를 띄울 때 아래의 코드 한 줄이면 된다.

```
loaded_model = mlflow.pyfunc.load_model(model_uri="models:/Semiconductor_Sentiment_FinBERT@production")
```


![MLexperiment](/photos/experiment0514.png)
