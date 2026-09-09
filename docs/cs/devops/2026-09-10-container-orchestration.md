# 컨테이너 오케스트레이션

- 여러 컨테이너의 **배포, 확장, 복구, 네트워크, 설정**을 자동화한다.
- 대표 도구는 Kubernetes이며, 선언적 설정으로 원하는 상태를 정의하고 실제 상태를 지속적으로 맞춘다.
- 현업에서는 무중단 배포, 트래픽 증가 대응, 장애 자동 복구, 개발·스테이징·운영 환경 표준화에 활용한다.

## 개념 설명

컨테이너 하나를 직접 실행하는 방식은 초기에는 간단하지만, 서비스 수가 늘어나면 어느 서버에 배치할지, 장애 컨테이너를 어떻게 재시작할지, 트래픽을 어떻게 분배할지 관리하기 어렵다. 오케스트레이터는 이 문제를 클러스터 단위로 해결한다.

Kubernetes를 예로 들면 **Pod**는 실행 단위이고, **Deployment**는 애플리케이션의 복제본 수와 업데이트 전략을 관리한다. **Service**는 변하지 않는 네트워크 주소와 로드밸런싱을 제공하며, **Ingress**는 외부 HTTP 요청을 여러 서비스로 라우팅한다. ConfigMap과 Secret은 환경 설정 및 민감 정보를 분리한다.

현업에서는 이미지가 레지스트리에 등록되면 CI/CD가 Deployment를 갱신한다. Kubernetes는 새 ReplicaSet을 만들고 준비 상태를 확인하면서 점진적으로 교체한다. 새 버전의 오류율이 증가하면 이전 버전으로 롤백할 수 있다. CPU·메모리 요청과 제한을 명시하면 스케줄러가 적절한 노드에 배치하고, HPA는 CPU나 요청량에 따라 Pod 수를 자동 조절한다.

다만 오케스트레이션은 운영 복잡도도 높인다. 로그·메트릭·트레이싱을 중앙화하고, 네트워크 정책, RBAC, Secret 관리, 비용 및 장애 도메인을 함께 설계해야 한다. 작은 서비스에는 관리형 Kubernetes나 서버리스 컨테이너가 더 적합할 수 있다.

## 실무 예시: 무중단 롤링 업데이트

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  selector:
    matchLabels: {app: api}
  template:
    metadata:
      labels: {app: api}
    spec:
      containers:
      - name: api
        image: registry.example.com/api:1.4.0
        readinessProbe:
          httpGet: {path: /health, port: 8080}
```

`readinessProbe`가 성공한 컨테이너만 트래픽을 받으므로, 애플리케이션 기동이 끝나기 전에 요청이 유입되는 문제를 줄인다. `replicas: 3`은 일부 Pod가 교체되는 동안에도 서비스를 유지하는 기반이 된다.

## 배포 흐름

```mermaid
flowchart LR
    A["코드 커밋"] --> B["CI 이미지 빌드"]
    B --> C["컨테이너 레지스트리"]
    C --> D["Deployment 갱신"]
    D --> E["새 Pod 생성"]
    E --> F["Readiness 검사"]
    F --> G["Service 트래픽 전환"]
```

## 면접 질문

### 1. Deployment와 Service의 차이는?

**답변:** Deployment는 Pod의 배포 상태, 복제본 수, 롤링 업데이트와 롤백을 관리한다. Service는 Pod가 교체되어도 동일한 주소로 접근하도록 디스커버리와 로드밸런싱을 제공한다.

### 2. 컨테이너가 계속 재시작되는 원인과 확인 방법은?

**답변:** 애플리케이션 예외, 잘못된 환경 변수, 메모리 제한 초과, 헬스 체크 실패가 주요 원인이다. `kubectl describe pod`, `kubectl logs --previous`, 이벤트와 종료 코드, 리소스 메트릭을 순서대로 확인한다.

> **한 줄 정리:** 컨테이너 오케스트레이션은 선언적 배포와 자동 복구·확장을 통해 분산 서비스 운영을 표준화한다.
