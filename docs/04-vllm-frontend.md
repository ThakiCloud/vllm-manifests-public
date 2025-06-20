# vllm-frontend: 추가 프론트엔드

`vllm-frontend` 디렉토리는 vLLM 스택을 위한 추가적인 프론트엔드 애플리케이션을 배포하기 위한 매니페스트를 포함합니다. 이는 `vllm-nextchat`의 NextChat UI와는 별개의 또 다른 사용자 인터페이스일 수 있습니다.

## 디렉토리 구조

```
vllm-frontend/
├── frontend-deployment.yaml # 프론트엔드 애플리케이션의 Deployment
└── frontend-pvc.yaml        # 프론트엔드 데이터 저장을 위한 PVC (필요시)
```

## 배포 방법

이 컴포넌트는 표준 Kubernetes 매니페스트로 구성되어 있으므로 `kubectl`을 사용하여 배포합니다.

1.  **네임스페이스 생성**:
    프론트엔드를 배포할 네임스페이스를 생성합니다. (예: `vllm-frontend`)

    ```bash
    kubectl create namespace vllm-frontend
    ```

2.  **매니페스트 적용**:
    `kubectl apply` 명령어를 사용하여 매니페스트를 클러스터에 적용합니다.

    ```bash
    kubectl apply -n vllm-frontend -f vllm-frontend/
    ```

## 주요 리소스 설명

-   `frontend-deployment.yaml`: 프론트엔드 컨테이너 이미지, 레플리카 수, 포트 설정 등을 정의하는 Deployment 리소스입니다. 어떤 컨테이너 이미지를 사용할지는 이 파일 안의 `spec.template.spec.containers[0].image` 필드를 확인하고 필요시 수정해야 합니다.
-   `frontend-pvc.yaml`: 프론트엔드 애플리케이션이 상태나 데이터를 영구적으로 저장해야 할 경우 사용하는 PersistentVolumeClaim입니다. 모든 프론트엔드에 필요한 것은 아닙니다.

배포 후에는 `kubectl get pods -n vllm-frontend`로 파드 상태를 확인하고, Ingress 설정을 추가하여 외부에서 접근할 수 있도록 해야 합니다. Ingress 설정은 `nginx-ingress/ingress.yaml` 파일에 새로운 규칙을 추가하여 관리할 수 있습니다. 