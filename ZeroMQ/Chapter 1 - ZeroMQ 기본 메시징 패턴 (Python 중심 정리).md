

> ZeroMQ의 핵심 메시징 패턴 3가지를 Python 코드 중심으로 설명합니다.  
> 각 패턴은 실전에서 어떻게 동작하며 어떤 상황에 유용한지까지 함께 설명합니다.

---

## 📌 다룰 내용

- Request-Reply (요청-응답) [[Chapter 3 - Advanced Request-Reply Patterns (고급 요청-응답 패턴)]]
- Publish-Subscribe (발행-구독)
- Pipeline (작업 분산)

---

## 🧩 공통 개념 요약

### 🔑 핵심 개념

- ZeroMQ는 전통적인 소켓처럼 보이지만, 내부적으로 **비동기 메시지 큐**로 동작합니다.
- 네트워크 프로그래밍에서의 **바이트 스트림(TCP)** 이 아닌, **메시지 단위로 전송**합니다.
- `zmq.Context()`는 ZeroMQ의 실행 환경을 설정하며, 각 소켓은 이 context 안에서 생성됩니다.
- 패턴마다 소켓 타입이 다르며, 이에 따라 메시지를 주고받는 방식이 완전히 달라집니다.

---

## 🔁 1. Request-Reply Pattern (REQ-REP)

> ✅ 클라이언트가 요청(request)을 보내고, 서버가 응답(reply)을 반환하는 구조입니다.  
> ✅ HTTP, RPC, RESTful API 등과 유사한 방식입니다.  
> ✅ 요청은 반드시 응답을 받아야 하며, 요청/응답 순서가 1:1로 고정됩니다.

---

### 🐍 Python 코드

```python
# server.py
import zmq

ctx = zmq.Context()
socket = ctx.socket(zmq.REP)
socket.bind("tcp://*:5555")

while True:
    message = socket.recv()
    print("[서버] 클라이언트로부터:", message.decode())
    socket.send(b"World")
```

```python
# client.py
import zmq

ctx = zmq.Context()
socket = ctx.socket(zmq.REQ)
socket.connect("tcp://localhost:5555")

socket.send(b"Hello")
reply = socket.recv()
print("[클라이언트] 서버 응답:", reply.decode())
```

---

### 🧠 보충 설명

- 클라이언트는 `REQ`, 서버는 `REP` 소켓을 사용합니다.
- `REQ`는 반드시 `send()` 후 `recv()`를 호출해야 하며, 순서를 지키지 않으면 동작하지 않습니다.
- `REP`은 그 반대로 `recv()` 후 `send()` 순서로 동작합니다.
- 단순한 질의/응답 API나 명령어 처리 시스템에 적합합니다.

---

## 📡 2. Publish-Subscribe Pattern (PUB-SUB)

> ✅ 발행자(PUB)가 메시지를 브로드캐스트 하면, 구독자(SUB)는 그 중 일부를 선택해서 수신합니다.  
> ✅ 완전히 **단방향 전송**이며, 구독자는 응답을 보낼 수 없습니다.  
> ✅ 뉴스 전송, 상태 알림, 실시간 로그 전송 등에 유용합니다.

---

### 🐍 Python 코드

```python
# publisher.py
import zmq
import time

ctx = zmq.Context()
socket = ctx.socket(zmq.PUB)
socket.bind("tcp://*:5556")
time.sleep(1)  # 구독자가 연결될 시간 확보

socket.send_string("weather.snow -10")
```

```python
# subscriber.py
import zmq

ctx = zmq.Context()
socket = ctx.socket(zmq.SUB)
socket.connect("tcp://localhost:5556")
socket.setsockopt_string(zmq.SUBSCRIBE, "weather")  # 필터 설정

msg = socket.recv_string()
print("[구독자] 메시지 수신:", msg)
```

---

### 🧠 보충 설명

- `PUB` 소켓은 구독자 수와 무관하게 메시지를 전송합니다.
- `SUB` 소켓은 `setsockopt_string()`으로 필터를 설정해야 메시지를 수신할 수 있습니다.
- 아무런 필터를 설정하지 않으면 **아무 메시지도 수신하지 않음**에 주의하세요.
- 메시지는 보장되지 않으며, 늦게 연결된 구독자는 **이미 전송된 메시지를 받지 못합니다**.

---

## 🏭 3. Pipeline Pattern (PUSH-PULL)

> ✅ 작업을 여러 worker에게 분산시킬 수 있는 구조입니다.  
> ✅ 전형적인 fan-out 구조이며, worker는 PUSH한 메시지를 하나씩 가져갑니다.  
> ✅ 비동기 작업 처리, 로그 수집, 분산 처리에 자주 사용됩니다.

---

### 🐍 Python 코드

```python
# ventilator.py (작업 분배자)
import zmq

ctx = zmq.Context()
socket = ctx.socket(zmq.PUSH)
socket.bind("tcp://*:5557")

for i in range(5):
    task = f"작업-{i}"
    socket.send_string(task)
```

```python
# worker.py (작업 수신자)
import zmq

ctx = zmq.Context()
socket = ctx.socket(zmq.PULL)
socket.connect("tcp://localhost:5557")

while True:
    msg = socket.recv_string()
    print("[워커] 작업 수신:", msg)
```

---

### 🧠 보충 설명

- `PUSH`는 데이터를 여러 `PULL` 워커에게 **자동으로 분산**합니다.
- 하나의 메시지는 단 하나의 워커에게만 전달되며, 로드밸런싱은 내부적으로 처리됩니다.
- 워커 간 중복이 없으며, 동시에 여러 워커가 실행돼도 문제없이 병렬 처리됩니다.
- 대표적인 용도: 비동기 작업 큐, 로그 파이프라인, 이미지 처리 클러스터

---

## 📌 패턴 요약 비교

| 패턴        | 구조       | 메시지 방향  | 소켓 타입        | 용도 예시                  |
|-------------|------------|----------------|------------------|-----------------------------|
| REQ-REP     | 1:1        | 양방향         | `REQ`, `REP`     | RPC, API, 요청/응답 시스템   |
| PUB-SUB     | 1:N        | 단방향         | `PUB`, `SUB`     | 로그, 알림, 뉴스 방송        |
| PUSH-PULL   | 1:N (fan-out) | 단방향       | `PUSH`, `PULL`   | 워커풀, 병렬처리, 작업큐     |

---

## 📎 참고

- Python용 ZeroMQ 설치: `pip install pyzmq`
- 공식 튜토리얼 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)
