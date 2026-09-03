# Prefetching: 필요한 데이터를 미리 가져오는 최적화

- **핵심 목표:** 실제 사용 시점보다 먼저 데이터나 명령을 준비해 대기 시간을 숨긴다.
- **대표 적용처:** CPU 캐시, 데이터베이스 연관 데이터, 웹 브라우저의 페이지·이미지·API 요청.
- **주의점:** 예측이 틀리면 네트워크·메모리·캐시 공간을 낭비하고, 오히려 성능이 저하된다.

## 개념 설명

Prefetching은 프로그램이 곧 사용할 가능성이 높은 데이터를 사용 전에 미리 읽어 두는 기법이다. 데이터가 도착하면 CPU 캐시, 애플리케이션 메모리, 브라우저 캐시 등에 저장되어 실제 접근 시 지연 시간이 줄어든다.

CPU에서는 하드웨어가 순차 메모리 접근 패턴을 감지해 다음 캐시 라인을 자동으로 가져온다. 개발자가 C/C++의 `__builtin_prefetch` 같은 명령으로 힌트를 줄 수도 있지만, 잘못 사용하면 메모리 대역폭과 캐시를 오염시킨다.

현업에서는 데이터베이스 조회에서 자주 사용된다. 예를 들어 게시글 목록을 조회한 뒤 각 게시글의 작성자를 개별 조회하면 N+1 문제가 발생한다. ORM의 `select_related`, `prefetch_related`, JPA의 fetch join 등으로 연관 데이터를 한 번에 미리 조회하면 쿼리 수를 크게 줄일 수 있다. 다만 필요하지 않은 컬럼과 관계까지 가져오는 eager loading은 응답 크기와 DB 부하를 증가시킨다.

웹 서비스에서는 사용자가 다음에 이동할 가능성이 높은 페이지의 API·이미지를 미리 요청한다. 검색 결과에서 다음 페이지를 prefetch하거나, 링크에 `rel="prefetch"`를 지정하는 방식이 있다. 트래픽 비용이 큰 모바일 환경에서는 최근 사용 패턴, 네트워크 상태, 캐시 적중률을 기준으로 조건부 실행해야 한다.

좋은 prefetching은 **예측 정확도**, **데이터 신선도**, **자원 비용**, **취소 가능성**을 함께 고려한다. 성능 개선 전후로 p95 지연 시간, 캐시 적중률, 불필요한 요청 비율을 측정해야 한다.

## 코드 예시

```javascript
const cache = new Map();

async function getPage(page) {
  if (!cache.has(page)) {
    const response = await fetch(`/api/items?page=${page}`);
    cache.set(page, await response.json());
  }
  return cache.get(page);
}

async function showPage(page) {
  const data = await getPage(page);
  render(data);
  // 사용자가 다음 페이지로 이동할 가능성을 고려한 prefetch
  if (navigator.connection?.saveData !== true) getPage(page + 1);
}
```

## 동작 흐름

```mermaid
flowchart LR
    A["현재 페이지 요청"] --> B["응답 표시"]
    B --> C{"다음 페이지 가능성"}
    C -->|높음| D["다음 데이터 prefetch"]
    C -->|낮음| E["prefetch 생략"]
    D --> F["캐시에 저장"]
    F --> G["다음 요청 즉시 응답"]
```

## 면접 질문

### 1. Prefetching과 Caching의 차이는?

Caching은 이미 사용한 데이터를 저장해 재사용하는 전략이고, prefetching은 아직 사용하지 않았지만 곧 필요할 데이터를 미리 가져오는 전략이다. Prefetch한 데이터도 보통 캐시에 저장되므로 함께 사용된다.

### 2. Prefetching이 성능을 악화시킬 수 있는 경우는?

예측이 틀려 불필요한 요청이 늘거나, 메모리·캐시·네트워크 대역폭을 점유할 때다. 낮은 적중률, 큰 응답, 오래된 데이터, 모바일 네트워크에서는 조건부 적용과 TTL·취소 처리가 필요하다.

> **한 줄 정리:** Prefetching은 “미리 준비해 지연을 숨기는 기술”이지만, 예측 정확도와 자원 비용을 측정할 때만 효과적인 최적화다.
