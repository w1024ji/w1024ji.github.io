---
category: "Orderflow"
---

# Phase 0: Kafka 띄우는 데 하루 썼다

오늘 새 프로젝트를 시작했다. 실시간 암호화폐 호가 데이터를 처리하는 스트리밍 파이프라인인데, 이름은 일단 **OrderFlow**로 정했다. 첫 번째 할 일은 minikube 위에 Kafka를 올리는 것. "별거 있겠어, helm으로 후딱 깔면 되지" 했는데... 하루를 다 썼다. 뭘 만났는지 기록해둔다.

---

## 문제 1. Bitnami 이미지가 사라졌다

helm으로 Kafka를 설치했다. 명령은 잘 먹혔고 파드도 생겼는데, 상태가 이상했다.

```
NAME                 READY   STATUS                  RESTARTS   AGE
kafka-controller-0   0/1     Init:ImagePullBackOff   0          119s
kafka-controller-1   0/1     Init:ErrImagePull       0          119s
kafka-controller-2   0/1     Init:ImagePullBackOff   0          119s
```

`kubectl describe`로 이벤트를 보니 이런 에러가 찍혀 있었다.

```
Failed to pull image "docker.io/bitnami/kafka:latest":
manifest for bitnami/kafka:latest not found: manifest unknown: manifest unknown
```

처음엔 오타인 줄 알았다. 이미지 이름 다시 확인하고, 다시 깔아보고... 근데 계속 같은 에러였다.

원인은 **Bitnami가 2025년 8월 28일부로 Docker Hub에서 무료 이미지를 내렸기 때문**이었다. 예전엔 `docker.io/bitnami/kafka`에서 공짜로 받을 수 있었는데, 이제 Broadcom이 Bitnami를 유료 모델(`Bitnami Secure Images`)로 전환하면서 대부분의 이미지가 `docker.io/bitnamilegacy`로 이동하거나 아예 사라졌다. 그래서 `bitnami/kafka:latest`는 manifest 자체가 없는 거였다.

해결책은 **`bitnamilegacy` 레포로 이미지를 바꿔주는 것**이었다. helm upgrade로 이미지 repository를 교체하면 된다. 다만 `bitnamilegacy`는 Bitnami가 공식적으로 "임시용"이라고 못박은 레포라 보안 패치가 없고 언젠가 내려갈 수도 있다. 그래도 로컬 개발 4개월짜리에는 충분하다고 판단했다. 이미 받은 이미지는 minikube에 캐시되니까 클러스터를 통째로 날리지 않는 한 계속 동작한다.

---

## 문제 2. `image.tag`가 `<nil>`이 됐다

이미지 레포를 바꾸는 과정에서 `--reuse-values`랑 `--reset-values`를 번갈아 쓰다가 실수가 생겼다. `kubectl get pods`로 확인해보니 파드 3개 중 2개는 멀쩡히 떴는데 1개만 이상한 상태였다.

```
NAME                 READY   STATUS                  RESTARTS   AGE
kafka-controller-0   1/1     Running                 0          3m2s
kafka-controller-1   1/1     Running                 0          3m2s
kafka-controller-2   0/1     Init:InvalidImageName   0          86s
```

이미지 이름을 찍어보니 이렇게 나왔다.

```
prepare-config=docker.io/bitnamilegacy/kafka:<nil>
kafka=docker.io/bitnamilegacy/kafka:<nil>
```

콜론 뒤가 `<nil>` — 즉 태그가 비어버린 거였다. `--reset-values`로 기존 값을 싹 날렸는데 `image.tag`를 명시적으로 안 줬더니 차트가 기본값을 못 채우고 빈 값으로 렌더한 것이다.

0번과 1번 파드가 멀쩡했던 건, 그 파드들은 이전 리비전(`:latest`가 있던 시절)의 스펙을 물고 있었기 때문이다. StatefulSet이 2→1→0 역순으로 파드를 업데이트하면서 2번만 새 스펙(태그 없음)을 받은 것. 만약 그대로 뒀으면 0번, 1번도 재생성될 때 결국 같은 문제가 생겼을 거다.

해결은 간단했다. **태그를 명시적으로 박는 것.** 실제로 0번 파드가 받았던 이미지 태그를 확인하고(`4.0.0-debian-12-r10`) 그걸 명시해서 다시 적용했다. StatefulSet 스펙에 제대로 박혔는지 확인한 다음, 파드 전체를 강제 재생성해서 3개가 전부 같은 스펙으로 맞췄다.

**교훈:** helm upgrade할 때 `--reset-values`를 쓰면 기존 커스텀 값이 전부 날아간다. 이미지 태그처럼 "차트 기본값이 있겠지" 싶은 것도 명시적으로 줘야 한다. 특히 Bitnami 계열은 이 부분이 좀 예민하다.

---

## 문제 3. SASL 인증이 켜져 있었다

3개 파드가 전부 `Running 1/1`이 됐다. 이제 토픽 만들고 produce/consume 테스트만 하면 Phase 0이 끝나는 상황.

```bash
kubectl exec -it kafka-controller-0 -n data-pipeline -- \
  kafka-topics.sh --bootstrap-server localhost:9092 --list
```

그런데 이 명령이 아무 응답 없이 그냥 멈춰버렸다. 30초를 기다렸는데도 반응이 없어서 Ctrl+C로 끊었다.

로그를 보니 원인이 나왔다.

```
InvalidRequestException: Unexpected Kafka request of type METADATA during SASL handshake.
```

**Bitnami Kafka 차트가 기본으로 client 인증(SASL/SCRAM)을 켜놓은 것**이었다. 브로커는 "SASL 핸드셰이크부터 해야지"라고 기다리는데, 내가 보낸 `kafka-topics.sh`는 인증 정보 없이 그냥 METADATA 요청을 날리니 브로커가 연결을 끊는 거였다. 그래서 명령이 무한 재시도에 빠진 것.

인증을 넣어서 붙는 방법도 있지만, 여기서 방향을 결정해야 했다. 로컬 개발 단계에서 인증을 켜두면 Producer, Flink 등 이후에 붙는 모든 컴포넌트마다 인증 설정을 달아야 해서 마찰이 커진다. 반대로 나중에 인증을 추가하는 과정 자체를 의도적인 작업으로 다루면, 오히려 "평문 → 인증 적용" 흐름을 보여줄 수 있어서 포트폴리오 서사로도 더 낫다.

그래서 **지금은 인증을 끄고, Phase 7 하드닝 단계에서 다시 켜는 것**으로 결정했다.

```bash
helm upgrade kafka oci://registry-1.docker.io/bitnamicharts/kafka \
  -n data-pipeline --reuse-values \
  --set listeners.client.protocol=PLAINTEXT
```

파드를 재생성하고 다시 테스트했다. 이번엔 깔끔하게 됐다.

---

## 최종 확인

produce/consume 테스트까지 통과했다.

```bash
# produce
echo "hello-orderflow" | kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test

# consume
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning --timeout-ms 10000

# 출력
hello-orderflow
Processed a total of 1 messages
```

Phase 0 완료. minikube, Kafka 3노드(KRaft), Prometheus + Grafana, 네임스페이스 구성까지 다 됐다.

---

## 정리

생각보다 오래 걸렸지만 건진 게 있다.

- **Bitnami 무료 이미지 종료** — 2025년 8월 이후 새로 시작하는 프로젝트라면 이 문제를 거의 무조건 만난다. `bitnamilegacy`로 우회하거나 아예 다른 이미지로 갈아타는 걸 처음부터 염두에 두는 게 낫다.
- **helm upgrade할 때 태그 명시** — `--reset-values`는 조심해서 써야 한다. 이미지 태그 같은 건 항상 명시적으로.
- **SASL 켜져 있는지 확인** — Bitnami Kafka 차트는 기본으로 인증이 켜져 있다. 클라이언트가 그냥 매달리면 SASL 문제일 가능성이 높다.

다음 포스트는 Phase 1 — Binance WebSocket으로 오더북 데이터를 수신하고, 스냅샷과 diff를 조합해서 오더북을 재구성하는 Producer 구현이다.
