# nginx-ingress: 외부 트래픽 관리

`nginx-ingress` 디렉토리는 Kubernetes 클러스터 외부의 트래픽을 내부 서비스로 라우팅하는 NGINX Ingress Controller 관련 매니페스트를 포함합니다. 사용자가 웹 브라우저나 API 클라이언트를 통해 서비스에 접근할 수 있도록 진입점(Entrypoint) 역할을 합니다.

## 디렉토리 구조

```
nginx-ingress/
├── ingress-service.yaml # Ingress Controller를 위한 LoadBalancer 서비스
├── ingress.yaml         # 실제 Ingress 리소스 (규칙 정의)
└── tcp-forwarding.yaml  # TCP 포트 포워딩을 위한 ConfigMap (필요시 사용)
```

## 배포 방법

NGINX Ingress Controller는 클러스터 전체에 영향을 미치는 핵심 컴포넌트이므로, 일반적으로 한 번만 설치합니다.

### 1. NGINX Ingress Controller 설치 (선행 작업)

이 리포지토리의 매니페스트를 적용하기 전에, 클러스터에 NGINX Ingress Controller가 먼저 설치되어 있어야 합니다. 공식 Helm 차트나 매니페스트를 사용하여 설치하는 것을 권장합니다.

```bash
# NGINX Ingress Controller 공식 Helm 차트로 설치하는 예시
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.extraArgs.tcp-services-configmap="nginx-ingress/tcp-forwarding"
```

### 2. Ingress 규칙 적용

Controller가 준비되면, 이 디렉토리의 매니페스트를 적용하여 실제 라우팅 규칙을 설정합니다.

```bash
# ingress-nginx 네임스페이스에 리소스 배포
kubectl apply -n ingress-nginx -f nginx-ingress/
```

### 3. TCP 서비스 포워딩 (선택 사항)

HTTP/HTTPS 외에 다른 TCP 프로토콜 기반의 서비스를 외부로 노출시켜야 할 때 TCP 포워딩을 사용합니다. 예를 들어, SSH나 데이터베이스에 직접 접근해야 할 경우 유용합니다.

Helm 설치 명령어에 포함된 `--set controller.extraArgs.tcp-services-configmap="nginx-ingress/tcp-forwarding"` 옵션은 NGINX Ingress Controller가 `nginx-ingress` 네임스페이스에 있는 `tcp-forwarding`이라는 이름의 ConfigMap을 TCP 설정 소스로 사용하도록 지정합니다.

**설정 방법:**

1.  **`nginx-ingress/tcp-forwarding.yaml` 파일 수정**:
    이 ConfigMap의 `data` 필드에 `<외부 포트>: "<네임스페이스>/<서비스 이름>:<서비스 포트>"` 형식으로 항목을 추가하거나 수정합니다.

    ```yaml
    # 예시: tcp-forwarding.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: tcp-forwarding
      namespace: nginx-ingress
    data:
      # 외부 포트 10001번으로 들어온 요청을
      # vllm-nextchat 네임스페이스의 vllm 서비스의 8000번 포트로 전달
      "10001": "vllm-nextchat/vllm:8000"
    ```

2.  **`nginx-ingress/ingress-service.yaml` 파일 수정**:
    `tcp-forwarding.yaml`에 추가한 외부 포트를 LoadBalancer 서비스에 노출시켜야 합니다. `spec.ports` 목록에 해당 포트를 추가합니다.

    ```yaml
    # 예시: ingress-service.yaml
    # ...
    spec:
      ports:
      # ... (기존 http, https 포트)
      - name: vllm # 포트 이름
        port: 10001  # 외부 포트
        protocol: TCP
    # ...
    ```

3.  **매니페스트 적용**:
    수정한 파일을 클러스터에 적용합니다.
    ```bash
    kubectl apply -f nginx-ingress/tcp-forwarding.yaml
    kubectl apply -f nginx-ingress/ingress-service.yaml

    kubectl rollout restart deployment nginx-ingress-ingress-nginx-controller -n nginx-ingress
    ```

### 주요 리소스 설명

-   `ingress-service.yaml`: Ingress Controller Pod를 외부에 노출시키기 위한 `LoadBalancer` 타입의 서비스를 정의합니다. 클라우드 환경에서는 외부 IP를 할당받게 됩니다.
-   `ingress.yaml`: `vllm-nextchat`이나 `grafana` 같은 내부 서비스들을 특정 호스트 이름(e.g., `chat.example.com`)과 경로(e.g., `/`)에 따라 어떻게 연결할지 정의하는 핵심 규칙 파일입니다.
-   `tcp-forwarding.yaml`: HTTP/HTTPS 외에 특정 TCP 포트(e.g., 5432 for PostgreSQL)를 포워딩해야 할 때 사용하는 ConfigMap입니다.

배포가 완료되면 `kubectl get svc -n ingress-nginx` 명령어로 외부 IP를 확인하고, DNS 설정을 통해 해당 IP로 도메인을 연결할 수 있습니다. 