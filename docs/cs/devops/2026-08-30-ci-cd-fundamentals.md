# CI/CD fundamentals

- **CI(Continuous Integration)**: 개발자가 변경 사항을 자주 통합하고, 자동 빌드·테스트로 결함을 빠르게 발견한다.
- **CD(Continuous Delivery/Deployment)**: 검증된 결과물을 배포 가능한 상태로 만들거나, 운영 환경까지 자동 배포한다.
- 핵심은 자동화보다 **작은 변경, 빠른 피드백, 재현 가능한 배포, 실패 시 롤백**이다.

## 개념 설명

CI/CD 파이프라인은 보통 `Checkout → Build → Test → Package → Deploy` 순서로 실행된다.  
CI는 코드가 저장소에 push되거나 Pull Request가 생성될 때 동작하며, 컴파일 오류와 단위 테스트 실패를 병합 전에 차단한다. 따라서 테스트가 없는 자동화는 CI라기보다 단순 빌드 자동화에 가깝다.

CD에는 두 가지 의미가 있다. **Continuous Delivery**는 운영 배포 직전까지 자동화하고 실제 배포는 승인 후 수행하는 방식이다. **Continuous Deployment**는 모든 검증을 통과하면 운영까지 자동 배포한다. 조직의 장애 허용 수준과 승인 정책에 따라 선택한다.

실무에서는 테스트 피라미드, 린트, 보안 취약점 검사, 환경별 설정 분리, 배포 이력 기록이 중요하다. 비밀번호와 토큰은 저장소에 넣지 않고 CI 플랫폼의 Secret에 보관한다. 배포 실패에 대비해 이전 이미지 버전으로 되돌릴 수 있어야 하며, 데이터베이스 스키마 변경은 구버전 애플리케이션과도 호환되도록 단계적으로 적용한다.

아래 예시는 GitHub Actions에서 Node.js 프로젝트의 CI와 조건부 배포를 구성한 것이다. `main` 브랜치 push만 배포하도록 제한하고, 테스트가 실패하면 배포 단계가 실행되지 않는다.

```yaml
name: ci-cd
on:
  pull_request:
  push:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm test -- --runInBand
      - run: npm run build
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: ./scripts/deploy.sh
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
```

## 파이프라인 흐름

```mermaid
flowchart LR
    A["코드 Push"] --> B["빌드"]
    B --> C["테스트"]
    C --> D["이미지 생성"]
    D --> E["스테이징 배포"]
    E --> F["승인 또는 자동화"]
    F --> G["운영 배포"]
```

## 면접 질문

### 1. CI와 CD의 차이는 무엇인가요?

CI는 변경 사항을 자주 통합하면서 빌드와 테스트를 자동 수행하는 과정이다. CD는 검증된 결과물을 배포 가능한 상태로 만들거나 운영 환경까지 자동 배포하는 과정이다.

### 2. 배포 후 장애가 발생하면 어떻게 대응하나요?

모니터링과 헬스 체크로 장애를 감지한 뒤, 이전 이미지나 릴리스 버전으로 롤백한다. 이후 로그와 메트릭으로 원인을 분석하고, 재발 방지 테스트를 파이프라인에 추가한다.

> **한 줄 정리:** CI/CD는 코드를 자주 통합하고 자동 검증·배포하여 릴리스 속도와 안정성을 함께 높이는 개발 운영 방식이다.
