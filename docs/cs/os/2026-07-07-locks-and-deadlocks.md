# Locks와 Deadlocks 핵심 정리

- **Lock**은 공유 자원에 대한 **동시 접근을 제어**해 데이터 경쟁(race condition)을 막는다.
- **Deadlock**은 둘 이상의 스레드가 서로의 자원을 기다리며 **영원히 멈추는 상태**다.
- 실무에서는 **락 순서 통일**, **타임아웃**, **tryLock** 같은 전략으로 예방한다.

## Concept explanation

락은 임계 구역(Critical Section)을 보호한다. 예를 들어 Java에서는 `synchronized`나 `ReentrantLock`을 사용한다.

```java
class Counter {
    private int value = 0;

    public synchronized void inc() {
        value++;
    }
}
```

위 코드는 한 번에 한 스레드만 `inc()`를 실행하게 해 `value++`의 비원자적 문제를 막는다.

반면 데드락은 락을 **서로 다른 순서로** 잡을 때 자주 발생한다.

```java
class DeadlockExample {
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    void method1() {
        synchronized (lockA) {
            synchronized (lockB) {
                System.out.println("A -> B");
            }
        }
    }

    void method2() {
        synchronized (lockB) {
            synchronized (lockA) {
                System.out.println("B -> A");
            }
        }
    }
}
```

`method1()`이 `lockA`를 잡고 `lockB`를 기다리는 동안, `method2()`는 `lockB`를 잡고 `lockA`를 기다릴 수 있다. 이때 둘 다 멈춘다.

예방 포인트:
```java
// 1) 항상 같은 순서로 락 획득
// 2) tryLock으로 실패 시 재시도
// 3) 락 범위를 최소화
```

## Short example or analogy

은행 창구와 번호표를 생각하면 된다.  
락은 **한 창구를 한 사람만 사용**하게 하는 것, 데드락은 **A 창구 번호표를 가진 사람과 B 창구 번호표를 가진 사람이 서로의 창구만 기다리는 상황**이다.

## Interview Questions

**Q1. Lock과 Deadlock의 차이는?**  
A1. Lock은 안전한 동시성을 위한 제어 수단이고, Deadlock은 락 사용이 꼬여 스레드가 서로 대기만 하는 장애 상태다.

**Q2. Deadlock을 어떻게 예방하나?**  
A2. 락 획득 순서를 통일하고, 필요 시 `tryLock()`이나 timeout을 사용하며, 동시에 잡는 락 수를 줄인다.

## One-line takeaway

**락은 경쟁을 막고, 데드락은 락 설계를 잘못했을 때 생기는 정지 상태다.**
