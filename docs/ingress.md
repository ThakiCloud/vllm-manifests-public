# Ingress 배포 가이드

## 개요
이 문서는 `ingress/` 디렉토리의 리소스를 사용하여 Kubernetes 클러스터의 외부 트래픽을 내부 서비스로 라우팅하는 방법을 설명합니다. Ingress-NGINX 컨트롤러를 사용하는 것을 기준으로 합니다.

## 포함된 매니페스트
- `ingress.yaml`: NextChat UI와 Grafana 대시보드로의 접근을 정의하는 핵심 Ingress 리소스입니다.

## 배포 전 요구사항
- Kubernetes 클러스터에 Ingress-NGINX 컨트롤러가 설치되어 있어야 합니다.
- 라우팅에 사용할 도메인의 DNS가 Ingress 컨트롤러의 외부 IP 주소로 설정되어 있어야 합니다.

## 배포 절차

1.  **호스트 이름 수정**:
    `ingress/ingress.yaml` 파일을 열어 `host` 필드의 값을 실제 사용할 도메인 이름으로 변경합니다.

    ```yaml
    # ingress/ingress.yaml

    # ... (생략) ...
    spec:
      rules:
        - host: "chat.your-domain.com" # <-- 이 부분을 실제 도메인으로 변경
          http:
    # ... (생략) ...
    ```

2.  **매니페스트 배포**:
    아래 명령어를 사용하여 Ingress 리소스를 클러스터에 배포합니다.

    ```bash
    kubectl apply -f ingress/
    ```

## 배포 후 확인
배포가 완료되면 설정한 호스트 이름(예: `http://chat.your-domain.com`)으로 접속하여 NextChat UI가 정상적으로 표시되는지 확인합니다. 또한, `/grafana` 경로(예: `http://chat.your-domain.com/grafana`)로 접속하여 Grafana 로그인 페이지가 나타나는지 확인합니다.

## 주요 설정
- **호스트 (`host`)**: 사용자의 외부 접속 도메인입니다.
- **경로 (`path`)**:
    - `/`: NextChat 서비스로 라우팅됩니다.
    - `/grafana`: Grafana 서비스로 라우팅됩니다.
- **서비스 이름 (`service.name`)**: Ingress가 트래픽을 전달할 대상 서비스의 이름입니다. (`nextchat`, `grafana-service`)
- **서비스 포트 (`service.port.number`)**: 대상 서비스의 포트 번호입니다. 