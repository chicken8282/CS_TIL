# Consistency Models 정리

- **일관성 모델**은 분산 시스템에서 “어느 시점에 어떤 값이 보이는가”를 정의한다.
- 강한 일관성일수록 읽기 결과 예측이 쉽지만, 지연과 가용성 비용이 커진다.
- 면접에서는 **CAP, RYW(Read Your Writes), eventual consistency**와 함께 묻는 경우가 많다.

## Concept explanation

일관성 모델(consistency model)은 여러 노드에 데이터가 복제된 환경에서, **쓰기(write) 후 읽기(read)가 어떤 순서로 관측되는지**에 대한 약속이다.  
대표적으로:

- **Strong consistency / Linearizability**: 쓰기가 성공하면 모든 이후 읽기에서 즉시 최신 값이 보인다.
- **Sequential consistency**: 모든 요청이 하나의 전역 순서로 보이지만, 실제 시간 순서와 반드시 일치하지는 않는다.
- **Eventual consistency**: 더 이상 쓰기가 없으면 시간이 지나 결국 모든 복제본이 같은 값을 가진다.
- **Read-your-writes**: 내가 방금 쓴 값은 이후 내가 읽을 때는 보장된다. 세션 일관성의 일부로 자주 언급된다.

강한 일관성은 사용자 경험이 직관적이지만, 분산 환경에서는 동기화 비용이 높아 **지연 증가** 또는 **가용성 저하**가 생길 수 있다. 반대로 eventual consistency는 빠르고 확장성이 좋지만, 잠깐 동안 **옛 값(stale read)** 을 볼 수 있다.

## Short example or analogy

예: SNS 프로필 이름을 바꿨다.  
- **Strong consistency**면 바로 모든 기기에서 새 이름이 보인다.  
- **Eventual consistency**면 잠시 동안 일부 기기에서는 예전 이름이 보일 수 있다.  
비유하면, 강한 일관성은 “모든 사람에게 동시에 공지”이고, eventual consistency는 “시간차를 두고 소식이 퍼지는 것”이다.

## Interview questions

**Q1. 강한 일관성과 최종 일관성의 차이는?**  
A. 강한 일관성은 쓰기 직후 모든 읽기가 최신 값을 보장하지만, 최종 일관성은 시간이 지나면 같아질 것만 보장한다.

**Q2. 왜 분산 시스템에서 강한 일관성이 비싼가?**  
A. 여러 노드가 같은 순서를 보게 하려면 합의, 동기화, 리더 선출 같은 비용이 필요해 지연과 장애 민감도가 커진다.

## Takeaway

**일관성 모델은 “정답이 언제 보이는가”에 대한 약속이며, 시스템 설계는 강한 정합성과 성능/가용성의 균형 문제다.**
