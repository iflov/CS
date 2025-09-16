
# ✅ Chapter 4 -  Advanced Pub-Sub Patterns (고급 발행-구독 패턴)

> 기본 PUB-SUB에서 확장된 구조인 **XPUB-XSUB**와 **중간 브로커(proxy)**를 통한 Pub-Sub 라우팅을 다룹니다.

---

## 📌 다룰 내용

- PUB-SUB 구조의 한계
- XPUB-XSUB 소켓 소개
- subscribe/unsubscribe 감지
- 중간 브로커(proxy) 구조
- 필터링 메커니즘 이해
- 사용자 정의 토픽 구조 설계

---

## 📡 PUB-SUB의 구조 요약

### 기본 구조

```text
Publisher --> Subscriber
```

- **PUB**: 발행자. 브로드캐스트만 수행. 수신 확인 없음.
- **SUB**: 구독자. `setsockopt`으로 원하는 토픽을 필터링.
- 기본 구조에서는 구독자가 무엇을 구독 중인지 **Publisher는 알 수 없음**.

---

## 🧱 XPUB & XSUB 소켓

> PUB/SUB을 확장한 소켓으로 **구독 이벤트(subscribe/unsubscribe)**를 감지하고, **중간 브로커 역할**을 할 수 있도록 설계됨.

| 소켓 타입 | 설명                             |
|-----------|----------------------------------|
| `XPUB`    | PUB의 확장. SUB의 구독 상태를 수신 |
| `XSUB`    | SUB의 확장. PUB으로부터 수신함     |

---

## 🧪 Subscribe 이벤트 감지 (XPUB)

### 🐍 Python 코드

```python
import zmq

ctx = zmq.Context()
xpub = ctx.socket(zmq.XPUB)
xpub.bind("tcp://*:5556")

while True:
    msg = xpub.recv()
    if msg[0] == 1:
        print("📬 구독 요청:", msg[1:].decode())
    elif msg[0] == 0:
        print("📭 구독 해제:", msg[1:].decode())
```

### 💡 설명

- XPUB는 구독자가 `setsockopt(zmq.SUBSCRIBE, topic)` 호출할 때 구독 이벤트를 수신합니다.
- 첫 바이트: `0x01`은 구독 요청, `0x00`은 구독 해제
- 이후 바이트: 구독하고자 하는 토픽 문자열

---

## 🔁 중간 브로커 구성 (proxy)

> 중간에 **XPUB-XSUB 소켓 쌍**을 넣어서 **여러 Publisher/Subscriber를 프록시**하는 구조

```text
Publisher --> [XSUB] --- (proxy) --- [XPUB] --> Subscriber
```

### 🐍 Python 코드

```python
import zmq

ctx = zmq.Context()

frontend = ctx.socket(zmq.XSUB)
frontend.bind("tcp://*:5555")  # Publisher 연결

backend = ctx.socket(zmq.XPUB)
backend.bind("tcp://*:5556")   # Subscriber 연결

zmq.proxy(frontend, backend)
```

### 💡 설명

- `zmq.proxy(xsub, xpub)`는 자동 라우팅 프록시를 구성합니다.
- 브로커는 subscribe/unsubscribe 메시지를 받아서 **토픽 기반으로 메시지를 라우팅**합니다.
- 여러 개의 publisher/subscriber를 한 곳에 모아 분산 구조를 쉽게 구성할 수 있음

---

## 🧵 SUB 소켓의 필터링

### 🐍 Python 코드

```python
ctx = zmq.Context()
sub = ctx.socket(zmq.SUB)
sub.connect("tcp://localhost:5556")
sub.setsockopt_string(zmq.SUBSCRIBE, "sports")  # "sports"로 시작하는 토픽만 수신
```

### 💡 설명

- 구독 필터는 문자열 prefix 매칭입니다.
- `"sports"`를 구독하면 `"sports.football"`, `"sports.basketball"` 등 모두 수신
- 필터를 빈 문자열로 설정하면 **모든 메시지를 수신** (`""` → 전체 구독)

---

## 🧪 다중 토픽 구독

```python
sub.setsockopt_string(zmq.SUBSCRIBE, "weather.")
sub.setsockopt_string(zmq.SUBSCRIBE, "finance.")
```

➡️ 여러 토픽을 동시에 구독할 수 있습니다.  
➡️ 각각의 필터는 **OR 조건**으로 동작합니다.

---

## 🧹 Unsubscribe 사용

```python
sub.setsockopt_string(zmq.UNSUBSCRIBE, "weather.")
```

➡️ 특정 토픽 수신을 중단하고 싶을 때 사용합니다.

---

## 🧠 토픽 설계 전략

> 필터가 prefix 기반이라는 점을 활용하여 **의미 있는 계층 구조 설계** 가능

예시:

- `"weather.kr.seoul"`  
- `"weather.us.ny"`  
- `"finance.stock.kospi"`  
- `"finance.crypto.btc"`

➡️ 이를 통해 특정 지역, 특정 자산군, 특정 분야만 필터링 수신 가능

---

## 🚧 XPUB Verbose 모드

```python
xpub.setsockopt(zmq.XPUB_VERBOSE, 1)
```

➡️ 이 설정을 하면 XPUB는 **구독 이벤트 메시지를 자동 전달**  
➡️ SUB가 subscribe 시 그 메시지를 XPUB가 수신해 처리할 수 있음

---

## 📌 구조 요약

| 구성 요소 | 소켓 타입 | 역할                         |
|-----------|-----------|------------------------------|
| Publisher | `PUB`     | 메시지 발행 (단방향)           |
| Subscriber | `SUB`    | 토픽 기반 메시지 수신          |
| XSUB      | `XSUB`    | PUB 역할 프록시                |
| XPUB      | `XPUB`    | SUB 역할 프록시 및 필터 라우팅 |

---

## 📦 예시 아키텍처

```text
          [Publisher 1]     [Publisher 2]
                 │               │
              [XSUB]  ← 중간 브로커 → [XPUB]
                                    │
                           [Subscriber 1~N]
```

- 하나의 브로커를 통해 수많은 퍼블리셔/서브스크라이버가 통신 가능
- 메시지 라우팅은 브로커가 필터 기반으로 처리함

---

## 📝 참고

- `pyzmq` 설치: `pip install pyzmq`
- `zmq.proxy()`는 Python에서도 기본 지원됨
- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)