# vllm-nextchat: Core LLM Service

`vllm-nextchat` 디렉토리는 vLLM 추론 서버와 NextChat 웹 UI를 함께 배포하기 위한 Helm 차트로 구성되어 있습니다. 이 컴포넌트는 사용자와 직접 상호작용하는 핵심 LLM 채팅 서비스를 제공합니다.

## 디렉토리 구조

```
vllm-nextchat/
├── Chart.yaml          # Helm 차트 정보
├── custom-values.yaml  # 사용자 정의 설정 파일 (핵심)
└── charts/             # 의존성 있는 하위 차트
    ├── nextchat/       # NextChat UI 컴포넌트
    └── vllm/           # vLLM 서버 컴포넌트
```

## 핵심 설정 파일

-   **`custom-values.yaml`**: 이 파일은 **가장 중요한 설정 파일**입니다. 배포하려는 LLM 모델, GPU 리소스, 레플리카 수, Hugging Face 토큰, NextChat 설정 등 모든 주요 파라미터를 이 파일에서 수정합니다. 배포 전 반드시 이 파일을 자신의 환경에 맞게 검토하고 수정해야 합니다.

## 배포 방법

Helm을 사용하여 배포하는 것을 권장합니다.

### 방법 1: Helm으로 배포 (권장)

1.  **네임스페이스 생성**:
    ```bash
    kubectl create namespace vllm-nextchat
    ```

2.  **`custom-values.yaml` 수정**:
    `vllm-nextchat/custom-values.yaml` 파일을 열어 사용할 모델, 리소스, `hfToken` 등 환경에 맞는 값을 설정합니다.

3.  **Helm 설치 또는 업그레이드**:
    `vllm-nextchat` 디렉토리에서 아래 명령어를 실행하여 Helm 릴리스를 설치하거나 업그레이드합니다.

    ```bash
    # VLLM 배포
    helm install vllm ./charts/vllm \
    --namespace vllm-nextchat \
    --set vllm.model="Qwen/Qwen3-8B" \
    --set hfToken.enabled=true \
    --set hfToken.token="$HF_TOKEN" \
    --values custom-values.yaml \
    --timeout 600s
    
    # NextChat 배포
    helm install nextchat ./charts/nextchat \
    --namespace vllm-nextchat \
    --set nextchat.baseUrl="http://vllm:8000" \
    --values custom-values.yaml
    ```

### 방법 2: `kubectl`로 렌더링된 매니페스트 배포

Helm을 사용하지 않고 순수 Kubernetes 매니페스트로 배포하고 싶다면, `helm template` 명령어로 매니페스트를 생성한 후 `kubectl`로 적용할 수 있습니다.

1.  **매니페스트 생성**:
    ```bash
    helm template vllm-chat . \
      -n vllm-nextchat \
      -f custom-values.yaml > rendered-manifest.yaml
    ```

2.  **매니페스트 적용**:
    ```bash
    kubectl apply -f rendered-manifest.yaml
    ```

## 주요 설정 (`custom-values.yaml`)

-   `vllm.model`: Hugging Face에서 사용할 LLM 모델 이름 (예: `EleutherAI/polyglot-ko-12.8b`)
-   `vllm.hfToken`: Private 모델 다운로드를 위한 Hugging Face 토큰 Secret의 이름
-   `vllm.resources`: vLLM 파드에 할당할 GPU, CPU, 메모리 리소스
-   `nextchat.customModels`: NextChat UI에 표시될 모델 목록 정의
-   `ingress.enabled`: Ingress 리소스 생성 여부
-   `ingress.hosts`: Ingress에 연결할 도메인 주소

배포 후 `kubectl get pods -n vllm-nextchat` 명령으로 파드들이 정상적으로 실행되는지 확인합니다. 