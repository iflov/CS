# ✅ Chapter 12: High-Water Mark(HWM)와 메시지 손실 방지 전략

> ZeroMQ는 내부적으로 메시지를 큐에 저장합니다.  
> 이때 큐의 최대 용량을 초과하면 **메시지를 버릴지(block할지)** 결정해야 하는데,  
> 이를 제어하는 것이 **High-Water Mark (HWM)** 입니다.

---

## 📌 다룰 내용

- HWM이란?
- HWM 기본 동작 방식
- 메시지 손실 vs 블로킹 제어
- HWM 설정 방법
- sender / receiver의 HWM 분리
- 실전 메시지 처리 전략

---

## 💡 HWM이란?

> HWM(High-Water Mark)은 **ZeroMQ 내부 큐의 최대 메시지 수 제한**입니다.

- 소켓 입출력 큐는 **비동기 전송을 위해 버퍼**를 사용
- HWM에 도달하면 다음 중 하나 발생:
  - 송신자: `send()`가 **block** 되거나 메시지를 **drop**
  - 수신자: 큐가 가득 차면 송신자가 block됨

---

## 🔧 기본 설정값

| 옵션        | 기본값 (pyzmq) |
|-------------|----------------|
| `SNDHWM`    | 1000           |
| `RCVHWM`    | 1000           |

- 1000개 메시지가 큐에 쌓이면 block/dropped
- 상황에 따라 값을 조정하여 대기/손실 여부를 제어 가능

---

## 🛠 Python에서 HWM 설정

```python
socket.setsockopt(zmq.SNDHWM, 5000)
socket.setsockopt(zmq.RCVHWM, 5000)
```

- **송신 큐 최대 5000개 메시지**
- **수신 큐 최대 5000개 메시지**

---

## 🧪 송신자에서 메시지 drop 여부 확인

```python
import zmq

ctx = zmq.Context()
sock = ctx.socket(zmq.PUSH)
sock.setsockopt(zmq.SNDHWM, 10)  # 작은 큐
sock.setsockopt(zmq.LINGER, 0)   # 종료 시 메시지 즉시 버림
sock.connect("tcp://localhost:5555")

for i in range(100):
    try:
        sock.send(f"msg {i}".encode(), zmq.NOBLOCK)
    except zmq.Again:
        print(f"❌ 메시지 드롭됨: msg {i}")
```

- 큐가 가득 차면 `zmq.Again` 예외 발생
- `NOBLOCK` 플래그가 없으면 블로킹됨

---

## 📥 수신자 HWM 설정

```python
sock = ctx.socket(zmq.PULL)
sock.setsockopt(zmq.RCVHWM, 1000)
```

- 수신자가 메시지를 너무 느리게 처리하면 송신자의 큐가 밀릴 수 있음
- 수신 속도와 큐 크기를 잘 맞추어야 함

---

## 🧠 송수신자 쌍의 흐름 제어 전략

| 상황                         | 전략                             |
|------------------------------|----------------------------------|
| 느린 수신자                  | 수신자 RCVHWM 증가               |
| 빠른 송신자 → 유실 방지 필요 | SNDHWM 증가 or 블로킹 사용       |
| 유실 허용 & 성능 우선        | NOBLOCK 사용 + 메시지 드롭 감수   |
| 서버 종료 시 지연 방지       | `LINGER = 0` 설정                 |

---

## 🧪 송신자가 대기하지 않고 drop 하게 만들기

```python
sock.setsockopt(zmq.SNDHWM, 10)

for i in range(100):
    try:
        sock.send_string(f"msg {i}", flags=zmq.NOBLOCK)
    except zmq.Again:
        print("Drop:", i)
```

➡️ 블로킹을 원하지 않을 경우 NOBLOCK을 꼭 사용  
➡️ 대신 메시지 손실이 발생할 수 있음

---

## 🧵 LINGER와의 조합

```python
socket.setsockopt(zmq.LINGER, 0)
```

- 소켓 종료 시 대기 없이 큐 제거  
- `LINGER = -1` → 기본값 (전부 전송될 때까지 대기)

---

## 📊 메시지 처리량 실험 예시

```python
# Producer
for i in range(100000):
    try:
        socket.send(f"task {i}".encode(), zmq.NOBLOCK)
    except zmq.Again:
        dropped += 1

print("드롭된 메시지 수:", dropped)
```

➡️ HWM 값을 조절하며 시스템의 처리 한계를 측정 가능

---

## 📎 요약

| 항목         | 설명                                       |
|--------------|--------------------------------------------|
| HWM이란?      | 송수신 큐의 최대 메시지 수                 |
| 기본값        | 1000                                       |
| 설정 방법      | `.setsockopt(zmq.SNDHWM, N)` 등           |
| 블로킹 여부    | 기본은 block, `NOBLOCK` 시 예외 발생       |
| 메시지 유실 방지 | HWM 증가 + 블로킹 or 확인 로직 추가         |
| 성능 우선 시    | 드롭 허용 + 모니터링 + 빠른 소비 구조 필요  |

---

## 📝 참고

- HWM은 ZeroMQ의 흐름 제어 핵심 도구입니다.
- 메시지를 절대 유실하면 안 되는 경우, 반드시 HWM을 높이거나 `NOBLOCK`을 피하세요.
- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)