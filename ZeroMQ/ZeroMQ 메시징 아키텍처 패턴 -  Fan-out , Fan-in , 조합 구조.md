# ✅ ZeroMQ 메시징 아키텍처 패턴: Fan-out / Fan-in / 조합 구조

---

## 📌 개요

ZeroMQ를 활용한 메시징 시스템에서 자주 쓰이는 3가지 핵심 구조:

1. **Fan-out**: 하나 → 여러 수신자 (브로드캐스트 또는 분산)
2. **Fan-in**: 여러 송신자 → 하나의 수신자 (수렴)
3. **Fan-out + Fan-in**: 작업 분산 후 결과 수렴

이 문서에서는 각 구조를 **구조 그림, 설명, 코드, 유스케이스**로 정리합니다.

---

## 📤 1. Fan-out 구조

### 🧠 개념

> **하나의 송신자(Sender)**가 **여러 수신자(Receivers)**에게 메시지를 전파하는 구조

### 📊 구조도

```text
         [Sender]
            │
   ┌────────┴────────┐
   ↓        ↓        ↓
[Receiver1][Receiver2][Receiver3]
```

---

### ✅ ZeroMQ 예시 ① – PUB → SUB (브로드캐스트)

```python
# Publisher
pub = ctx.socket(zmq.PUB)
pub.bind("tcp://*:5555")
pub.send_string("news: update")

# Subscriber
sub = ctx.socket(zmq.SUB)
sub.connect("tcp://localhost:5555")
sub.setsockopt_string(zmq.SUBSCRIBE, "news")
msg = sub.recv_string()
```

- 모든 `SUB`가 같은 메시지를 받음
- 주로 **알림, 실시간 브로드캐스트**에 사용

---

### ✅ ZeroMQ 예시 ② – PUSH → PULL (분산 fan-out)

```python
# PUSH
push = ctx.socket(zmq.PUSH)
push.bind("tcp://*:6000")
push.send_string("task 1")

# PULL (여러 워커 중 하나가 받음)
pull = ctx.socket(zmq.PULL)
pull.connect("tcp://localhost:6000")
msg = pull.recv_string()
```

- 여러 워커 중 **한 명만** 작업을 받음
- 주로 **로드 밸런싱, 작업 분산**에 사용

---

### 💡 유스케이스

| 분야              | 설명                         |
|-------------------|------------------------------|
| 실시간 채팅 전파   | PUB-SUB                      |
| 로그 스트리밍      | PUB-SUB                      |
| 작업 큐 분산       | PUSH-PULL                    |
| 이벤트 브로드캐스트 | PUB-SUB                      |

---

## 📥 2. Fan-in 구조

### 🧠 개념

> **여러 송신자(Senders)**가 **하나의 수신자(Collector)**로 메시지를 보내는 구조

### 📊 구조도

```text
[Sender1]    [Sender2]    [Sender3]
     │           │             │
     └────┬──────┴─────┬───────┘
              ↓
         [Collector]
```

---

### ✅ ZeroMQ 예시 – PULL ← PUSH

```python
# 여러 PUSH (Producer)
push1 = ctx.socket(zmq.PUSH)
push1.connect("tcp://localhost:6000")
push1.send_string("task from 1")

# 하나의 PULL (Collector)
pull = ctx.socket(zmq.PULL)
pull.bind("tcp://*:6000")
msg = pull.recv_string()
```

- 여러 생산자들이 동일한 수신자에게 데이터 전송
- 주로 **로그 수집, 센서 데이터 수집, 이벤트 집계**에 사용

---

### 💡 유스케이스

| 분야                  | 설명                           |
|-----------------------|--------------------------------|
| 로그 수집 서버         | 여러 서비스가 로그를 PUSH       |
| 센서 네트워크 집계      | 다양한 노드의 데이터를 수렴     |
| 분석 시스템 입력 모음    | 여러 source → 하나의 처리 파이프 |

---

## 🔁 3. Fan-out + Fan-in 구조 (작업 분산 + 결과 수렴)

^0778e3

### 🧠 개념

> 작업을 여러 worker에게 분산(fan-out)하고, 그 결과를 다시 하나의 수신자가 수렴(fan-in)하는 구조

### 📊 구조도

```text
        [Dispatcher]
              │
      ┌───────┴────────┐
      ↓       ↓        ↓
   [Worker1][Worker2][Worker3]
      │       │        │
      └──┬─────┴───────┬┘
         ↓
     [Result Collector]
```

---

### ✅ ZeroMQ 예시 – PUSH → PULL + PULL ← PUSH

- Dispatcher: PUSH 소켓
- Workers: PULL (받기), PUSH (결과 보내기)
- Collector: PULL (결과 받기)

```python
# Dispatcher
push = ctx.socket(zmq.PUSH)
push.bind("tcp://*:7000")

# Worker
recv = ctx.socket(zmq.PULL)
recv.connect("tcp://localhost:7000")

send = ctx.socket(zmq.PUSH)
send.connect("tcp://localhost:7001")

msg = recv.recv_string()
result = msg.upper()
send.send_string(result)

# Collector
collector = ctx.socket(zmq.PULL)
collector.bind("tcp://*:7001")
print(collector.recv_string())
```

---

### 💡 유스케이스

| 분야            | 설명                                  |
|-----------------|-----------------------------------------|
| 대규모 이미지 처리 | 작업 분산 + 결과 수렴                  |
| 병렬 연산        | CPU-intensive 작업 분산 후 결과 수집      |
| 분산 데이터 수집  | 파편화된 분석 결과 병합                   |

---

## 📎 정리 비교

| 구조             | 송신자 수 | 수신자 수 | 메시지 흐름 예시     | ZeroMQ 패턴     | 주요 용도                      |
|------------------|-----------|-----------|-----------------------|------------------|-------------------------------|
| Fan-out          | 1         | N         | PUB → SUB, PUSH → PULL | PUB-SUB, PUSH-PULL | 알림, 브로드캐스트, 작업 분산 |
| Fan-in           | N         | 1         | PUSH → PULL            | PUSH-PULL        | 집계, 수렴, 센서 데이터 수신   |
| Fan-out + Fan-in | 1         | N → 1     | PUSH → PULL + PUSH → PULL | 혼합형           | 분산 처리 → 결과 수집         |

---

## 📝 참고

- ZeroMQ의 socket 타입 조합을 통해 간단하게 대규모 메시징 토폴로지 구현 가능
- PUB-SUB은 브로드캐스트 기반, PUSH-PULL은 로드 밸런싱 기반
- `zmq.proxy()`를 이용하면 중간 브로커를 통해 복잡한 구조도 가능

📚 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)  
📚 Python API: [https://pyzmq.readthedocs.io](https://pyzmq.readthedocs.io)