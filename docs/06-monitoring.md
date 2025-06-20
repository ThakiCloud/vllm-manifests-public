# monitoring: 시스템 상태 관찰

`monitoring` 디렉토리는 Kubernetes 클러스터와 vLLM 서비스의 상태를 관찰하기 위한 모니터링 스택 매니페스트를 포함합니다. Prometheus, Grafana 및 다양한 Exporter들로 구성되어 시스템의 핵심 메트릭을 수집하고 시각화합니다.

## 디렉토리 구조

```
monitoring/
├── cadvisor.yaml           # cAdvisor (컨테이너 리소스 메트릭 수집)
├── dcgm-exporter.yaml      # NVIDIA DCGM Exporter (GPU 메트릭 수집)
├── grafana-dashboard.yaml  # Grafana 대시보드 설정 (ConfigMap)
├── grafana-simple.yaml     # Grafana 배포
├── kube-state-metrics.yaml # Kubernetes 오브젝트 상태 메트릭 수집
├── node-exporter.yaml      # 노드(서버) 하드웨어 및 OS 메트릭 수집
├── prometheus-rbac.yaml    # Prometheus를 위한 RBAC (권한) 설정
├── prometheus-rule.yaml    # Prometheus 알림 규칙
└── prometheus-simple.yaml  # Prometheus 배포
```

## 배포 방법

모니터링 스택은 클러스터 전역에서 사용되므로 보통 한 번만 설치합니다. 모든 컴포넌트를 같은 네임스페이스(예: `monitoring`)에 배포하는 것이 일반적입니다.

1.  **네임스페이스 생성**:
    ```bash
    kubectl create namespace monitoring
    ```

2.  **매니페스트 적용**:
    `monitoring` 디렉토리의 모든 매니페스트를 순서에 상관없이 적용합니다. `prometheus-rbac.yaml`이 먼저 적용되면 좋지만, 필수적이지는 않습니다.

    ```bash
    kubectl apply -n monitoring -f monitoring/
    ```

## 주요 컴포넌트 설명

-   **Prometheus (`prometheus-simple.yaml`)**:
    -   핵심 시계열 데이터베이스. 다른 Exporter로부터 메트릭을 주기적으로 수집(scrape)합니다.
    -   vLLM 서버나 다른 서비스에서 메트릭을 수집하려면 Prometheus 설정 파일(ConfigMap)에 scrape job을 추가해야 합니다.

-   **Grafana (`grafana-simple.yaml`)**:
    -   Prometheus가 수집한 데이터를 시각화하는 대시보드 도구입니다.
    -   `grafana-dashboard.yaml`은 vLLM GPU 사용량을 보여주는 예시 대시보드를 정의합니다.

-   **Exporters**:
    -   **DCGM Exporter**: NVIDIA GPU의 사용률, 메모리, 온도 등 상세한 GPU 메트릭을 수집하여 Prometheus가 가져갈 수 있도록 노출합니다. GPU 워크로드를 모니터링하는 데 필수적입니다.
    -   **Node Exporter**: 각 노드의 CPU, 메모리, 디스크, 네트워크 사용량 등 시스템 레벨 메트릭을 수집합니다.
    -   **kube-state-metrics**: Deployment, Pod, Service 등 Kubernetes API 오브젝트의 상태(예: 파드 개수, 준비 상태)를 메트릭으로 변환합니다.
    -   **cAdvisor**: 각 컨테이너의 리소스 사용량(CPU, 메모리 등)에 대한 메트릭을 제공합니다.

배포 후에는 Ingress 설정을 추가하여 Grafana 대시보드에 외부에서 접근할 수 있도록 설정해야 합니다. (예: `nginx-ingress/ingress.yaml` 파일 수정) 