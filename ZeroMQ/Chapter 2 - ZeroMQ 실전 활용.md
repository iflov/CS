
> 실 프로그램에서 ZeroMQ를 어떻게 응용하는지를 다루는 실습 중심 정리입니다.  
> Python 중심 예제와 보충 설명을 통해 실전 적용에 집중합니다.

---

## 📌 목차

- ZeroMQ 소켓의 생성과 사용
- 메시지 송수신 (Python)
- 비동기 I/O 처리
- 다중 소켓 처리
- 에러 및 인터럽트 관리
- 종료 및 메모리 누수 관리
- Multipart 메시지
- 메시지 포워딩 및 브로커 작성
- 멀티쓰레드 메시징
- HWM 설정

---

## 🧱 The Socket API

> ZeroMQ는 BSD 소켓 스타일의 API를 따르지만, 내부는 **비동기 메시지 큐** 기반으로 동작합니다.

### ✅ 주요 동작 방식 요약

| 기능           | Python 예시 함수                     |
|----------------|--------------------------------------|
| 소켓 생성/종료 | `ctx.socket()` / `socket.close()`     |
| 소켓 설정      | `socket.setsockopt()`                 |
| 네트워크 연결  | `socket.bind()` / `socket.connect()`  |
| 메시지 송수신  | `socket.send()` / `socket.recv()`     |

- `Context`는 ZeroMQ의 실행 환경으로, 소켓은 이 context에서 생성
- `send()`는 메시지를 **큐에 넣는 비동기 동작**이며, 실제 전송은 별도 스레드에서 이루어짐

---

## 🌐 소켓 연결

### 🐍 Python 코드

```python
import zmq

ctx = zmq.Context()
socket = ctx.socket(zmq.REP)

# 여러 endpoint에 바인딩 가능
socket.bind("tcp://*:5555")
socket.bind("inproc://somename")
```

### 💡 설명

- `bind()`는 주로 서버 역할, `connect()`는 클라이언트 역할을 수행합니다.
- 하나의 소켓은 여러 endpoint를 바인딩 할 수 있으며, 여러 개의 연결을 동시에 처리할 수 있습니다.
- ZeroMQ는 자동으로 연결을 수립하고 재접속까지 처리하므로 복잡한 연결 관리가 불필요합니다.

---

## 📤 메시지 송수신

### 🐍 Python 코드

```python
# 송신
socket.send(b"Hello ZeroMQ")

# 수신
msg = socket.recv()
print("Received:", msg.decode())
```

### 💡 설명

- 메시지는 **bytes** 단위로 전송되며, 문자열을 보낼 경우 `b""` 또는 `.encode()` 사용 필요
- `.recv()`는 기본적으로 블로킹 함수이며, non-blocking 모드는 옵션으로 설정 가능
- 수신한 메시지는 `.decode()`로 문자열 변환

---

## 🧵 I/O Threads 설정

### 🐍 Python 코드

```python
ctx = zmq.Context(io_threads=4)
```

### 💡 설명

- `io_threads`는 내부적으로 네트워크 입출력을 처리하는 백그라운드 스레드 수를 의미합니다.
- 대부분의 애플리케이션은 1개로 충분하지만, 고속 통신을 처리할 경우 늘릴 수 있습니다.
- **멀티 소켓/멀티 노드 구조**에서도 하나의 context를 공유할 수 있습니다.

---

## 🧭 메시징 패턴

### 🔑 주요 메시징 패턴

| 패턴            | 설명                                |
|------------------|-------------------------------------|
| Request-Reply     | 요청-응답, 클라이언트-서버 구조          |
| Pub-Sub           | 발행-구독, 브로드캐스트 구조             |
| Pipeline          | fan-out/fan-in 구조, 작업 분산 처리용     |
| Exclusive Pair    | 하나의 페어만 연결, 내부 스레드 간 통신 등 |

### 💡 설명

- 각 패턴은 **소켓 타입**으로 결정됩니다.
- `REQ-REP`, `PUB-SUB`, `PUSH-PULL` 등은 서로 **1:1 또는 1:N 구조**로 통신합니다.
- ZeroMQ는 소켓 타입만 맞추면 **자동으로 메시지를 라우팅**하므로 패턴 사용이 쉽습니다.

---

## ✉️ 메시지 다루기

### 🐍 Python 코드

```python
socket.send(b"sample message")

msg = socket.recv()
print("Message:", msg.decode())
```

### 💡 설명

- Python에서는 별도 메시지 구조체 없이 간단하게 바이트 단위 메시지를 전송합니다.
- C처럼 `zmq_msg_t`를 직접 다루지 않아도 되므로 간결합니다.
- 메시지를 재사용하거나 복사할 필요 없이, 필요한 만큼 새로 생성해서 보내면 됩니다.

---

## 📦 Multipart 메시지

### 🐍 Python 코드

```python
socket.send_multipart([b"Header", b"Body", b"Footer"])
parts = socket.recv_multipart()
for part in parts:
    print(part.decode())
```

### 💡 설명

- 여러 메시지 파트를 하나의 메시지처럼 전송할 수 있습니다.
- 각 파트는 `bytes` 형식이며, 순서가 유지됩니다.
- `.recv_multipart()`는 리스트 형태로 모든 파트를 수신합니다.
- 헤더/본문/메타 정보 등을 나눠서 보낼 때 유용합니다.

---

## 🔀 다중 소켓 처리

### 🐍 Python 코드 (zmq.Poller)

```python
poller = zmq.Poller()
poller.register(socket1, zmq.POLLIN)
poller.register(socket2, zmq.POLLIN)

sockets = dict(poller.poll())

if socket1 in sockets:
    msg1 = socket1.recv()
if socket2 in sockets:
    msg2 = socket2.recv()
```

### 💡 설명

- `zmq.Poller`를 이용하면 여러 소켓을 동시에 감시할 수 있습니다.
- 이벤트 루프나 멀티 입력 처리 시 유용합니다.
- `poll()`의 timeout을 조절하면 blocking 또는 non-blocking 동작 설정 가능

---

## 🧯 에러 및 인터럽트 처리

### 🐍 Python 예시

```python
import zmq
import signal
import sys

def shutdown(sig, frame):
    print("종료 중...")
    socket.close()
    context.term()
    sys.exit(0)

signal.signal(signal.SIGINT, shutdown)
```

### 💡 설명

- `Ctrl+C` 신호 등을 수신하면 안전하게 소켓과 context를 종료할 수 있도록 합니다.
- `context.term()`을 호출하면 해당 context 하의 모든 소켓이 종료됩니다.
- 무한 루프나 서버 실행 시 필수적으로 들어가야 하는 패턴입니다.

---

## 🧪 메모리 누수 방지

### 💡 설명

- Python에서는 수신 후 메시지를 수동 해제할 필요는 없습니다.
- 하지만 `recv()` 후 메시지를 더 이상 사용하지 않을 경우 참조 해제를 권장합니다.
- context는 명시적으로 `term()` 또는 `destroy()`로 종료해야 합니다.

---

## 🧷 High-Water Mark (HWM)

### 🐍 Python 코드

```python
socket.setsockopt(zmq.SNDHWM, 1000)
socket.setsockopt(zmq.RCVHWM, 1000)
```

### 💡 설명

- HWM(High-Water Mark)은 메시지 큐의 최대 길이를 설정하는 옵션입니다.
- 큐가 HWM을 넘어서면 메시지를 drop 하거나 block 할 수 있습니다.
- 대량의 메시지 전송 시 메모리 폭주를 방지하는 안전장치입니다.

---

## 📝 참고

- Python용 ZeroMQ 라이브러리 설치:  
  ```bash
  pip install pyzmq
  ```
- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)
- Python API 문서: [https://pyzmq.readthedocs.io](https://pyzmq.readthedocs.io)