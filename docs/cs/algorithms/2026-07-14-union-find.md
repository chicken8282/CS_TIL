# Union-Find(Disjoint Set)

- **서로소 집합 관리**: 여러 원소가 어떤 그룹에 속하는지 빠르게 추적한다.
- **연산 2개가 핵심**: `find`로 대표 원소를 찾고, `union`으로 두 집합을 합친다.
- **최적화가 중요**: 경로 압축(path compression)과 union by rank/size를 쓰면 거의 O(1)에 가깝게 동작한다.

## Concept explanation

Union-Find는 **집합을 트리 형태로 표현**하는 자료구조다. 각 원소는 부모를 가지며, 같은 집합의 최상단 대표 원소를 **루트(root)** 라고 한다.  
`find(x)`는 x의 루트를 찾는다. 이때 매번 부모를 루트로 갱신하는 **경로 압축**을 하면 이후 조회가 빨라진다.  
`union(a, b)`는 두 원소의 루트를 비교해 다른 집합이면 한쪽을 다른 쪽 아래에 붙인다. 보통 **랭크/사이즈가 작은 트리**를 큰 트리 밑에 붙여 높이를 줄인다.

실무/면접에서 자주 쓰이는 경우:
- **사이클 판별**: 무방향 그래프에서 간선을 추가할 때 두 정점의 루트가 같으면 사이클
- **네트워크 연결성**: 친구 관계, 섬 연결, 계정 그룹 병합
- **Kruskal MST**: 간선 선택 시 같은 집합인지 빠르게 체크

## Code example

```python
class UnionFind:
    def __init__(self, n):
        self.p = list(range(n))
        self.r = [0]*n

    def find(self, x):
        if self.p[x] != x:
            self.p[x] = self.find(self.p[x])
        return self.p[x]

    def union(self, a, b):
        a, b = self.find(a), self.find(b)
        if a == b: return False
        if self.r[a] < self.r[b]: a, b = b, a
        self.p[b] = a
        if self.r[a] == self.r[b]: self.r[a] += 1
        return True
```

```mermaid
flowchart TD
A[원소 x] --> B[find(x)]
B --> C{루트인가?}
C -- 아니오 --> D[부모를 따라감]
D --> B
C -- 예 --> E[대표 원소 반환]
F[union(a,b)] --> G[각 루트 찾기]
G --> H{같은 집합?}
H -- 아니오 --> I[작은 트리를 큰 트리에 연결]
```

## Interview questions

1. **경로 압축을 왜 쓰나요?**  
   `find`를 반복할수록 트리가 평평해져 전체 성능이 크게 좋아집니다.

2. **union by rank와 size의 차이는?**  
   rank는 높이 추정값, size는 집합 크기 기준입니다. 둘 다 트리 균형을 유지하는 목적입니다.

**Takeaway:** Union-Find는 “같은 그룹인지 빠르게 판단하고 합치는” 문제의 표준 해법이다.
