# Locks와 Deadlocks

- **Lock**은 여러 스레드가 같은 자원에 동시에 접근할 때 **임계 구역**을 보호한다.
- **Deadlock**은 둘 이상의 스레드가 서로의 락을 기다리며 **영원히 멈춘 상태**다.
- 실무에서는 **락 범위 최소화, 순서 통일, 타임아웃**이 핵심 대응이다.

## Concept explanation

락은 공유 자원의 일관성을 보장한다. 예를 들어 계좌 잔액을 수정할 때, 동시에 두 스레드가 읽고 쓰면 값이 꼬일 수 있다.

```python
import threading

balance = 1000
lock = threading.Lock()

def deposit():
    global balance
    with lock:
        temp = balance
        temp += 100
        balance = temp
```

위 코드에서 `with lock`은 임계 구역 진입을 직렬화한다. 락이 없으면 `balance`가 레이스 컨디션에 노출된다.

데드락은 보통 **상호 배제, 점유와 대기, 비선점, 순환 대기** 4조건이 동시에 만족될 때 발생한다.

```python
lock_a = threading.Lock()
lock_b = threading.Lock()

def task1():
    with lock_a:
        with lock_b:
            print("task1")

def task2():
    with lock_b:
        with lock_a:
            print("task2")
```

`task1`은 A를 잡고 B를 기다리고, `task2`는 B를 잡고 A를 기다리면 서로 영원히 막힌다.

## Short example or analogy

은행 창구를 생각하면 된다. 한 명이 번호표를 받고 창구를 쓰는 동안 다른 사람은 기다린다.  
그런데 A 창구 담당자가 B 문서를 기다리고, B 창구 담당자가 A 문서를 기다리면 둘 다 업무를 못 한다. 이것이 데드락이다.

실무 팁:
```python
# 항상 같은 순서로 락 획득
with lock_a:
    with lock_b:
        pass
```

```python
# 타임아웃으로 무한 대기 방지
acquired = lock_a.acquire(timeout=1)
if not acquired:
    raise TimeoutError()
```

## Interview questions

**Q1. 락과 뮤텍스는 같은가요?**  
A. 좁게 보면 둘 다 동기화 도구다. 실무에서는 보통 뮤텍스를 상호배제용 락으로 이해하면 된다.

**Q2. 데드락을 예방하는 방법은?**  
A. 락 획득 순서를 통일하고, 락 범위를 줄이며, 타임아웃이나 try-lock을 사용한다.

## One-line takeaway

**락은 안전을, 데드락은 정지를 만든다; 핵심은 “적게 잡고, 같은 순서로, 빨리 놓는 것”이다.**
