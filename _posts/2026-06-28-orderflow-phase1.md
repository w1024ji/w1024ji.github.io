---
category: "Orderflow"
---

# Phase 1: Binance 오더북 데이터를 Kafka에 흘려보내기

Phase 0에서 Kafka를 겨우 띄웠다. 이제 진짜 데이터를 흘릴 차례다.

Phase 1의 목표는 하나였다. **Binance에서 실시간 호가 데이터를 받아서, Avro로 직렬화하고, Kafka 토픽에 발행하는 Producer를 만드는 것.** 말로 하면 간단한데, 막상 해보니 개념적으로도 구현적으로도 생각보다 깊었다.

---

## 오더북(Orderbook)이란?

먼저 개념부터. 오더북은 거래소에서 현재 올라와 있는 **매수/매도 주문의 목록**이다.

- **매수 호가(Bids):** "나 이 가격에 살게요" 주문들. 높은 가격 순으로 정렬.
- **매도 호가(Asks):** "나 이 가격에 팔게요" 주문들. 낮은 가격 순으로 정렬.

예를 들면 이런 모양이다.

```
BIDS (매수)          ASKS (매도)
60,095.98  4.60     60,095.99  1.54
60,095.97  0.07     60,096.00  0.02
60,095.96  0.00     60,096.46  0.00
```

매수 최상단(60,095.98)과 매도 최상단(60,095.99) 사이의 간격을 **스프레드**라고 한다. 이 오더북이 실시간으로 계속 바뀌는 걸 받아서 처리하는 게 이 프로젝트의 핵심이다.

---

## Step 1. WebSocket으로 diff 스트림 받기

Binance는 오더북 데이터를 WebSocket으로 실시간 제공한다. 연결 주소는 이런 형태다.

```
wss://stream.binance.com:9443/ws/btcusdt@depth
```

붙어보면 데이터가 쏟아진다.

```
[1782464732014] bids=70 asks=68 | first bid: ['60139.99', '0.31278']
[1782464733014] bids=96 asks=47 | first bid: ['60139.99', '1.04084']
```

근데 여기서 중요한 게 있다. **이게 전체 오더북이 아니라 "변경분(diff)"이다.**

`bids=70`은 "지금 매수 호가가 70개다"가 아니라 "이번 이벤트에서 바뀐 매수 호가가 70개다"라는 뜻이다. 그리고 수량이 `0`으로 오면 그 가격대가 사라진 것이다. 이걸 모르고 그냥 쌓으면 완전히 틀린 오더북이 된다.

---

## Step 2. 오더북 재구성 — 가장 어려운 부분

그래서 진짜 오더북을 만들려면 **스냅샷 + diff 조합**이 필요하다. Binance 공식 문서에 나와있는 절차인데, 생각보다 까다롭다.

### 왜 순서가 중요한가

직관적으로 생각하면 "스냅샷 먼저 받고, 그 다음 diff 받으면 되는 거 아니냐"고 생각할 수 있다. 근데 그게 함정이다.

스냅샷을 REST API로 받는 동안 WebSocket에서 diff가 계속 날아온다. 스냅샷 받는 데 100ms가 걸렸다면, 그 사이에 온 diff를 놓치게 된다. 그러면 스냅샷 이후 오더북이 어긋난다.

올바른 순서는 이렇다.

1. **WebSocket 연결 먼저** — diff 스트림을 구독하고 버퍼에 쌓기 시작
2. **REST로 스냅샷 요청** — 현재 오더북 상태 + `lastUpdateId` 받기
3. **버퍼에 쌓인 diff 중 스냅샷보다 오래된 건 버리기**
4. **이후 diff를 순서대로 적용**

이 순서가 반드시 지켜져야 한다.

### 시퀀스 갭(Gap) 감지

Binance diff 이벤트에는 `U`(첫 번째 updateId)와 `u`(마지막 updateId)가 있다. 이전 이벤트의 `u + 1`이 다음 이벤트의 `U`여야 연속이다.

```
이벤트 1: U=100, u=105
이벤트 2: U=106, u=110  ← 연속 (정상)
이벤트 3: U=115, u=120  ← 갭 발생! 111~114가 없다
```

갭이 생기면 그 사이 변경분을 놓친 거라 오더북이 틀어진다. 이 경우 오더북을 버리고 처음부터 다시 동기화해야 한다. 이걸 코드로 구현하면:

```python
if first_update_id > self.last_update_id + 1:
    print(f"[!] GAP detected → resync")
    self.ready = False
    return False
```

이 갭 감지 로직이 없으면 오더북이 조용히 틀어지는데 알아채기가 어렵다. 금융 데이터에서 이런 조용한 오류가 제일 무섭다.

---

## Step 3. Avro + Schema Registry — 데이터 계약

오더북 데이터를 Kafka에 실을 때 JSON을 쓸 수도 있다. 근데 JSON은 문제가 있다.

- 필드 이름에 오타가 나도 모른다
- Consumer가 어떤 필드를 기대하는지 알 수 없다
- 나중에 필드가 추가/삭제되면 Producer와 Consumer가 동시에 바뀌어야 한다

이걸 해결하는 게 **Avro + Schema Registry**다.

**Avro**는 데이터 직렬화 포맷이다. JSON보다 훨씬 가볍고, 스키마가 강제된다. 스키마를 먼저 정의하고, 그 스키마에 맞지 않는 데이터는 직렬화 자체가 안 된다.

**Schema Registry**는 스키마를 중앙에서 관리하는 서버다. Producer가 스키마를 등록하면 ID를 받는다. 메시지에는 데이터 대신 그 ID만 실어서 보낸다. Consumer는 ID로 스키마를 조회해서 역직렬화한다. 이러면 스키마가 바뀌어도 하위 호환성을 관리할 수 있다.

오더북 이벤트 스키마는 이렇게 정의했다.

```json
{
  "type": "record",
  "name": "OrderBookEvent",
  "fields": [
    {"name": "symbol",         "type": "string"},
    {"name": "event_time",     "type": "long"},
    {"name": "last_update_id", "type": "long"},
    {"name": "bids",           "type": {"type": "array", "items": "PriceLevel"}},
    {"name": "asks",           "type": {"type": "array", "items": "PriceLevel"}}
  ]
}
```

그리고 Confluent wire format에 따라 메시지 앞에 magic byte(0x00) + schema ID(4 bytes)를 붙여서 Kafka에 실었다. Flink가 나중에 이걸 읽을 때 ID로 스키마를 조회해서 역직렬화할 수 있다.

---

## 문제 1. Kafka에 메시지가 안 들어갔다

오더북 재구성까지는 잘 됐다. 근데 Kafka에 발행하는 부분에서 막혔다.

```
Failed to resolve 'kafka-controller-0.kafka-controller-headless.data-pipeline.svc.cluster.local:9092'
알려진 호스트가 없습니다.
```

원인은 **Kafka가 자기 주소를 클러스터 내부 DNS로 광고(advertise)하기 때문**이었다.

Kafka는 클라이언트가 처음 붙으면 "나는 이 주소로 찾아와"라고 알려준다. 이걸 **advertised listener**라고 한다. Bitnami Kafka 차트는 이 주소를 파드의 헤드리스 서비스 FQDN(`kafka-controller-0.kafka-controller-headless.data-pipeline.svc.cluster.local`)으로 설정한다.

문제는 로컬 PC에서 Python을 돌리면 이 클러스터 내부 DNS를 못 푼다는 거다. `kubectl port-forward`로 9092를 열어도, 브로커가 "9092 말고 저 긴 주소로 와"라고 리다이렉트하니까 결국 못 붙는다.

advertised listener를 `localhost`로 덮어쓰려고 helm 설정도 바꿔봤는데, Bitnami 차트가 파드별로 동적으로 생성하는 구조라 이 방향으로는 계속 막혔다.

### 해결: Producer를 파드로 올리기

근본적인 해결책은 **Producer를 로컬에서 돌리지 않고 클러스터 안에 파드로 올리는 것**이었다. 파드 안에서는 클러스터 내부 DNS가 다 풀리니까 이 문제 자체가 사라진다.

어차피 최종 목표도 파드로 돌아가는 거였으니, 지금 Dockerfile을 짜고 파드로 올리는 게 맞는 방향이었다.

Dockerfile을 만들고, minikube에 직접 이미지를 빌드하고(`minikube image build`), K8s Deployment 매니페스트를 작성해서 올렸다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orderflow-producer
  namespace: data-pipeline
spec:
  replicas: 1
  selector:
    matchLabels:
      app: orderflow-producer
  template:
    metadata:
      labels:
        app: orderflow-producer
    spec:
      containers:
        - name: producer
          image: orderflow-producer:latest
          imagePullPolicy: Never
```

`imagePullPolicy: Never`는 레지스트리에서 이미지를 당기지 말고 minikube에 로컬로 빌드된 이미지를 쓰라는 뜻이다.

---

## 문제 2. minikube image build 경로 문제

Windows에서 `minikube image build`를 돌리니까 경로 변환 에러가 났다.

```
ERROR: failed to build: resolve : lstat /var/lib/minikube/build/build.229100208/C:: no such file or directory
```

minikube가 Windows 경로(`C:\...`)를 Linux 경로로 변환하다가 `C:`를 제대로 못 처리한 거다.

해결은 **`minikube ssh`로 VM 안에 직접 들어가서 빌드**하는 것이었다. `minikube cp`로 필요한 파일들을 VM으로 복사하고, SSH로 들어가서 `docker build`를 직접 돌렸다.

```bash
minikube ssh
cd /tmp/build
docker build -t orderflow-producer:latest .
```

---

## 최종 확인

파드 로그를 보니 이렇게 찍혔다.

```
[schema] registered id=1
[*] Connecting to Binance WebSocket: BTCUSDT
[snapshot] lastUpdateId=96461226783 | bids=1000 asks=1000
[*] Orderbook ready! Publishing to Kafka...
[1782467414014] id=96461227612 | bid=59837.00 ask=59837.01
[kafka] delivered → partition=0 offset=0
[1782467415014] id=96461228193 | bid=59837.00 ask=59837.01
[kafka] delivered → partition=0 offset=1
```

`[kafka] delivered → partition=0 offset=0` — 이게 찍히는 순간 Phase 1이 끝났다.

---

## 정리

Phase 1에서 배운 것들:

- **오더북 재구성은 순서가 전부다.** diff 버퍼링 → 스냅샷 → diff 적용 순서를 지키지 않으면 데이터가 조용히 틀어진다.
- **시퀀스 갭은 반드시 감지해야 한다.** 금융 데이터에서 조용한 오류가 제일 위험하다.
- **Avro + Schema Registry는 초반에 세팅하는 게 맞다.** 나중에 붙이려면 Producer/Consumer를 동시에 바꿔야 해서 더 번거롭다.
- **K8s 클러스터 안의 서비스는 클러스터 안에서 붙어야 한다.** 로컬에서 port-forward로 우회하는 건 단순한 테스트는 되지만, Kafka처럼 advertised listener가 있는 서비스는 결국 파드로 올리는 게 맞다.

다음은 Phase 2 — Flink로 1초 윈도우 연산이다. 오더북 데이터가 Kafka에 쌓이고 있으니, 이걸 읽어서 가중 호가 불균형 지표를 실시간으로 계산할 차례다.
