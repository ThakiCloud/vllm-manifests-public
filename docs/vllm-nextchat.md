# vLLM-NextChat 배포 가이드

## 개요
이 문서는 `vllm-nextchat/` 디렉토리에 포함된 Helm 차트를 사용하여 vLLM 추론 서버와 NextChat 웹 UI를 배포하는 방법을 설명합니다. 이 차트는 두 애플리케이션과 필요한 모든 Kubernetes 리소스를 함께 패키징하여 배포를 간소화합니다.

## 포함된 리소스
- `charts/`: vLLM과 NextChat의 Kubernetes 리소스(Deployment, Service, HPA, PVC 등)를 정의하는 Helm 템플릿 파일들이 위치합니다.
- `custom-values.yaml`: Helm 차트의 기본값을 재정의(override)하여 배포를 커스터마이징하는 데 사용되는 핵심 설정 파일입니다.
- `Chart.yaml` (암시적): 차트의 이름, 버전 등 메타데이터를 정의합니다.
- `templates/` (암시적): `custom-values.yaml`의 값을 참조하여 최종 Kubernetes 매니페스트를 생성하는 템플릿 파일들을 포함합니다.

## 배포 전 요구사항
- Kubernetes 클러스터에 Helm CLI가 설치되어 있어야 합니다.
- GPU 노드가 준비되어 있고, [NVIDIA Device Plugin](https://github.com/NVIDIA/k8s-device-plugin)이 설치되어 있어야 합니다.
- (필요시) Hugging Face 등 Private 모델 레지스트리 접근을 위한 시크릿이 준비되어 있어야 합니다.
- (필요시) 모델 및 캐시 저장을 위한 `StorageClass`가 정의되어 있어야 합니다.

## 배포 절차

1.  **네임스페이스 생성**:
    애플리케이션을 배포할 네임스페이스를 생성합니다.

    ```bash
    kubectl create namespace vllm-nextchat
    ```

2.  **`custom-values.yaml` 수정**:
    `vllm-nextchat/custom-values.yaml` 파일을 열어 배포 환경에 맞게 주요 값들을 수정합니다.

    **주요 설정 항목**:
    - `vllm.model`: 로드할 Hugging Face 모델 이름.
    - `vllm.nodeSelector`: vLLM Pod를 특정 GPU 노드에 스케줄링하기 위한 셀렉터.
    - `vllm.resources`: vLLM 컨테이너에 할당할 GPU, CPU, 메모리 리소스.
    - `nextchat.baseUrl`: NextChat이 통신할 vLLM의 내부 서비스 주소.
    - `persistence.storageClass`: 모델 다운로드를 위한 영구 볼륨의 스토리지 클래스.
    - `hpa.enabled`: 트래픽에 따른 자동 확장을 활성화할지 여부.

3.  **Helm을 사용하여 배포**:
    아래 명령어를 사용하여 `vllm-nextchat` Helm 차트를 배포합니다. `vllm-app`은 릴리스 이름이며, 원하는 대로 변경할 수 있습니다.

    ```bash
    helm install vllm-app vllm-nextchat/ -f vllm-nextchat/custom-values.yaml -n vllm-nextchat
    ```
    - `install`: 새로운 릴리스를 생성합니다.
    - `vllm-app`: 릴리스 이름입니다.
    - `vllm-nextchat/`: 차트의 경로입니다.
    - `-f vllm-nextchat/custom-values.yaml`: 커스텀 설정 파일을 지정합니다.
    - `-n vllm-nextchat`: 배포할 네임스페이스를 지정합니다.

## 배포 후 확인
- **Pod 상태 확인**: 배포 후 Pod들이 정상적으로 실행되는지 확인합니다.

    ```bash
    kubectl get pods -n vllm-nextchat -w
    ```
- **서비스 접속**: `ingress.yaml`이 배포되었다면 설정된 호스트 주소로 접속하여 NextChat UI를 확인합니다. Ingress가 없다면 `port-forward`를 사용하여 임시로 접속할 수 있습니다.

    ```bash
    # nextchat Pod 이름 확인
    kubectl get pods -n vllm-nextchat | grep nextchat
    
    # 포트포워딩 실행
    kubectl port-forward <nextchat-pod-name> 3000:3000 -n vllm-nextchat
    ```
    이후 웹 브라우저에서 `http://localhost:3000`으로 접속합니다.

## 업그레이드 및 삭제
- **업그레이드**: `custom-values.yaml` 변경 후에는 `helm upgrade` 명령어를 사용합니다.
    ```bash
    helm upgrade vllm-app vllm-nextchat/ -f vllm-nextchat/custom-values.yaml -n vllm-nextchat
    ```
- **삭제**: 배포된 릴리스를 삭제하려면 `helm uninstall` 명령어를 사용합니다.
    ```bash
    helm uninstall vllm-app -n vllm-nextchat
    ``` 