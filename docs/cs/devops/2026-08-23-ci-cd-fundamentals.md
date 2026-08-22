# CI/CD Fundamentals

## 핵심 요약

- **CI(Continuous Integration)**: 개발자가 코드를 자주 통합하고, 자동 빌드·테스트로 문제를 조기에 발견한다.
- **CD(Continuous Delivery/Deployment)**: 검증된 결과물을 배포 가능한 상태로 만들거나 운영 환경까지 자동 배포한다.
- 좋은 파이프라인은 **빠른 피드백, 재현성, 안전한 롤백**을 제공해야 한다.

## 개념 설명

CI/CD는 소스 코드 변경을 검증하고 배포하는 과정을 자동화하는 개발 방식이다. 일반적으로 개발자가 브랜치에 커밋을 push하면 CI 서버가 파이프라인을 실행한다. 파이프라인은 의존성 설치, 정적 분석, 테스트, 빌드 순서로 구성된다.

CI의 핵심은 작은 변경을 자주 통합하는 것이다. 테스트가 실패하면 병합을 막아 결함이 운영 환경으로 전달되는 것을 줄인다. 테스트는 단위 테스트뿐 아니라 린트, 타입 검사, 보안 검사까지 포함할 수 있다.

CD는 두 가지 의미로 사용된다. **Continuous Delivery**는 언제든 배포할 수 있도록 산출물을 준비하지만 운영 배포 승인 단계가 남아 있는 방식이다. **Continuous Deployment**는 모든 검증을 통과한 변경을 운영 환경에 자동 배포한다.

실무에서는 환경별 설정과 비밀 정보를 코드에 직접 저장하지 않고 CI 도구의 Secret 또는 외부 비밀 저장소를 사용한다. 또한 실패한 배포를 되돌릴 수 있도록 버전이 고정된 이미지, 아티팩트 저장소, 롤백 전략을 준비해야 한다. 파이프라인은 빠르게 유지하고, 오래 걸리는 통합 테스트는 병렬화하는 것이 좋다.

## 코드 예제

```yaml
name: CI

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
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --coverage
      - run: npm run build
```

## 실행 흐름

```mermaid
flowchart LR
    A["코드 push"] --> B["CI 실행"]
    B --> C["Lint 및 Test"]
    C --> D["Build"]
    D --> E["Artifact 저장"]
    E --> F["승인 또는 자동 배포"]
    F --> G["운영 모니터링"]
```

## 면접 질문

### 1. CI와 CD의 차이는 무엇인가요?

CI는 코드 변경을 자주 통합하고 자동 검증하는 과정입니다. CD는 검증된 결과물을 배포 가능한 상태로 유지하거나 운영 환경까지 자동 배포하는 과정입니다.

### 2. 배포 파이프라인이 실패했을 때 어떻게 대응하나요?

로그와 실패한 단계를 먼저 확인하고 원인을 수정합니다. 운영 장애라면 이전 안정 버전으로 즉시 롤백하며, 재발 방지를 위해 테스트·모니터링·배포 검증을 보완합니다.

> **한 줄 정리:** CI/CD는 코드 변경부터 검증과 배포까지를 자동화해 빠르고 안전한 릴리스를 가능하게 한다.
