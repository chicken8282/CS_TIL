# Connection Pooling이란?

- DB 연결을 요청할 때마다 새로 만들지 않고, 미리 만든 연결을 재사용하는 기술이다.
- 연결 생성·인증·해제 비용을 줄여 응답 시간을 낮추고 서버 자원을 절약한다.
- 풀 크기를 너무 작게 잡으면 대기하고, 너무 크게 잡으면 DB와 애플리케이션 모두 부담을 받는다.

## 개념 설명

DB 연결은 단순히 함수 하나를 호출하는 일이 아니다. 애플리케이션이 DB 서버를 찾고, TCP 연결을 맺고, 인증한 뒤 통신할 준비를 해야 한다. 요청이 끝난 뒤 연결을 매번 닫으면 다음 요청은 이 과정을 다시 반복한다.

Connection Pool은 **식당의 대기 테이블과 공용 식기**에 비유할 수 있다. 손님이 올 때마다 새 식기를 만들고 버리는 대신, 식기를 미리 준비해 두고 사용 후 세척하여 다시 사용한다. 여기서 식기는 DB 연결이고, 식기를 보관하는 곳이 Connection Pool이다.

애플리케이션은 풀에서 사용 가능한 연결을 빌려 SQL을 실행한 뒤 반드시 반환한다. 반환은 연결 자체를 물리적으로 종료한다는 뜻이 아니라, 다른 요청이 다시 사용할 수 있도록 풀에 돌려놓는다는 의미다.

풀에는 보통 최소 연결 수와 최대 연결 수가 있다. 요청이 적을 때는 최소 연결만 유지하고, 동시에 요청이 많아지면 최대 수까지 연결을 늘린다. 모든 연결이 사용 중이면 새 연결을 무작정 만들지 않고 잠시 기다린다. 대기 시간이 지나면 타임아웃 오류가 발생할 수 있다.

주의할 점은 연결을 반환하지 않는 누수다. 예외가 발생해도 반환되도록 `try-with-resources`나 프레임워크의 트랜잭션 관리를 사용해야 한다. 또한 풀의 최대 크기를 크게 설정한다고 무조건 빨라지지 않는다. DB가 처리할 수 있는 동시 작업 수를 초과하면 락 경합과 메모리 사용량이 증가한다.

## 코드 예시

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost/app");
config.setUsername("app");
config.setPassword("secret");
config.setMaximumPoolSize(10);
config.setConnectionTimeout(3000);

HikariDataSource pool = new HikariDataSource(config);

try (Connection conn = pool.getConnection();
     PreparedStatement ps = conn.prepareStatement(
         "SELECT * FROM users WHERE id = ?")) {
    ps.setLong(1, 1L);
    ResultSet rs = ps.executeQuery();
} // 연결이 닫힌 것처럼 보이지만 실제로는 풀에 반환됨
```

## 동작 흐름

```mermaid
flowchart LR
    A["애플리케이션 요청"] --> B["Pool에서 연결 획득"]
    B --> C["DB 쿼리 실행"]
    C --> D["연결 반환"]
    D --> E["다음 요청이 재사용"]
    B --> F["사용 가능한 연결 없음"]
    F --> G["대기 또는 타임아웃"]
```

## 면접 질문

1. **Connection Pool을 사용하는 이유는?**  
   매 요청마다 DB 연결을 생성·종료하는 비용을 줄이고, 연결을 재사용해 성능과 자원 효율을 높이기 위해서다.

2. **Pool 크기를 무작정 크게 하면 왜 안 되는가?**  
   DB의 동시 처리 한계를 넘겨 커넥션·메모리·락 경합이 증가하고, 오히려 성능이 저하될 수 있기 때문이다.

## 한 줄 정리

**Connection Pool은 DB 연결을 미리 준비해 빌려 쓰고 반환하는 재사용 창고다.**
