# Monitoring 스택 배포 가이드

## 개요
이 문서는 `monitoring/` 디렉토리의 매니페스트를 사용하여 Prometheus, Grafana, DCGM Exporter로 구성된 모니터링 스택을 배포하는 방법을 설명합니다.

## 포함된 매니페스트
- `prometheus-configmap.yaml`: Prometheus의 설정 파일입니다. Scrape 대상, 규칙 파일 경로 등을 정의합니다.
- `prometheus-deployment.yaml`: Prometheus 서버를 배포하는 Deployment 리소스입니다.
- `prometheus-service.yaml`: Prometheus 서버를 클러스터 내부에 노출하는 Service 리소스입니다.
- `prometheus-rule.yaml`: Prometheus에서 사용할 알림 규칙을 정의하는 PrometheusRule 리소스입니다.
- `grafana-simple.yaml`: Grafana Deployment, Service, Secret 및 기본 대시보드(ConfigMap)를 포함하는 간소화된 배포 파일입니다.
- `dcgm-exporter.yaml`: NVIDIA GPU 메트릭을 수집하는 DCGM Exporter를 DaemonSet으로 배포합니다.

## 배포 전 요구사항
- 클러스터의 GPU 노드에 NVIDIA 드라이버가 설치되어 있어야 합니다.
- (선택) PrometheusRule CRD를 사용하려면, 클러스터에 Prometheus Operator 또는 유사한 CRD가 설치되어 있어야 합니다. 만약 설치되어 있지 않다면 `prometheus-rule.yaml` 파일은 제외하고 배포해야 합니다.

## 배포 절차

1.  **네임스페이스 생성**:
    모니터링 컴포넌트를 격리된 네임스페이스에 배포하는 것을 권장합니다.

    ```bash
    kubectl create namespace monitoring
    ```

2.  **Grafana 비밀번호 설정 (권장)**:
    배포 전, `monitoring/grafana-simple.yaml` 파일 내 `grafana-secret` 리소스의 `admin-password` 값을 원하는 비밀번호의 Base64 인코딩 값으로 변경합니다.

    ```bash
    # 'your-secure-password'를 원하는 비밀번호로 변경하여 실행
    echo -n 'your-secure-password' | base64
    ```
    위 명령어로 생성된 값을 `grafana-simple.yaml`의 `admin-password` 필드에 붙여넣습니다.

3.  **매니페스트 배포**:
    아래 명령어를 사용하여 `monitoring/` 디렉토리의 모든 리소스를 배포합니다.

    ```bash
    kubectl apply -f monitoring/ -n monitoring
    ```

## 배포 후 확인
- **Grafana 접속**: `ingress.yaml`에 설정된 경로(예: `http://chat.your-domain.com/grafana`)로 접속하여 Grafana UI에 접근할 수 있는지 확인합니다. 초기 아이디는 `admin`이며, 비밀번호는 2단계에서 설정한 값을 사용합니다.
- **Prometheus 확인**: Prometheus UI에 직접 접근하려면 `port-forward`를 사용할 수 있습니다.
    ```bash
    # monitoring 네임스페이스의 prometheus-deployment-xxxx Pod 이름 확인
    kubectl get pods -n monitoring
    
    # 포트포워딩 실행
    kubectl port-forward <prometheus-pod-name> 9090:9090 -n monitoring
    ```
    이후 웹 브라우저에서 `http://localhost:9090`으로 접속하여 'Status' > 'Targets' 메뉴에서 `dcgm-exporter`와 같은 대상이 정상적으로 수집되고 있는지 확인합니다. 