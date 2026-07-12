# RISC vs CISC: 명령어 집합 설계의 차이

- **RISC**는 적은 수의 단순한 명령어를 빠르게 실행하도록 설계
- **CISC**는 복잡한 작업을 한 명령어로 처리하도록 설계
- 현대 CPU는 둘의 장점을 섞어 사용하며, **컴파일러 최적화**와 함께 봐야 함

## Concept explanation

### RISC (Reduced Instruction Set Computer)
- 명령어가 단순하고 길이가 일정한 경우가 많음
- load/store 구조가 핵심: 메모리 접근은 `load`, `store`로만 하고 연산은 레지스터에서 수행
- 파이프라인 구성에 유리해 고성능/저전력 설계에 자주 사용

```asm
; RISC 스타일 예시
LOAD R1, [A]
LOAD R2, [B]
ADD  R3, R1, R2
STORE [C], R3
```

### CISC (Complex Instruction Set Computer)
- 한 명령어가 더 많은 일을 수행
- 메모리 접근과 연산을 한 번에 처리하는 명령이 많음
- 코드 길이가 짧아질 수 있지만, 명령 해석이 복잡해질 수 있음

```asm
; CISC 스타일 예시
ADD [C], [A], [B]   ; 개념적으로 A + B를 C에 저장
```

### 핵심 비교
- RISC: `명령어 수↓`, `단순성↑`, `파이프라인↑`
- CISC: `명령어 수↓`, `명령어 복잡도↑`, `코드 밀도↑`

실제 개발에서는 어셈블리보다 **컴파일러가 생성한 기계어 패턴**이 중요하다. 예를 들어 같은 C 코드도 RISC 계열에서는 더 많은 명령어가 나올 수 있다.

```c
int c = a + b;
```

```asm
; RISC에서는 보통 분해됨
LOAD R1, [a]
LOAD R2, [b]
ADD  R3, R1, R2
STORE [c], R3
```

## Short example or analogy

- RISC는 **레고 블록**처럼 단순한 부품을 많이 조합하는 방식
- CISC는 **멀티툴**처럼 한 도구가 여러 기능을 맡는 방식

예:  
- RISC: `설정 → 계산 → 저장`을 각각 분리  
- CISC: 하나의 복합 명령으로 처리

## Interview questions

**Q1. RISC와 CISC의 가장 큰 차이는?**  
A. 명령어의 복잡도와 실행 방식이다. RISC는 단순 명령어 중심, CISC는 복합 명령어 중심이다.

**Q2. 왜 현대 CPU는 RISC/CISC를 단순 비교하면 안 되나?**  
A. 내부 구조는 복잡하지만 외부 ISA는 CISC여도, 내부적으로는 RISC처럼 분해해 실행하는 경우가 많기 때문이다.

## One-line takeaway

**RISC는 단순한 명령어로 빠른 실행을, CISC는 복잡한 명령어로 높은 코드 밀도를 노린다.**
