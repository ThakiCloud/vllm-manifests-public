# argocd-project: GitOps Entrypoint

`argocd-project` 디렉토리는 전체 LLM 서비스 스택을 GitOps 방식으로 배포하고 관리하기 위한 ArgoCD Application 매니페스트를 포함합니다. 각 YAML 파일은 특정 컴포넌트(예: 모니터링, vLLM-NextChat)를 클러스터에 배포하는 방법을 정의합니다.

## 디렉토리 구조

```
argocd-project/
├── frontend-argocd.yaml       # vLLM 프론트엔드 ArgoCD 애플리케이션
├── ingress-argocd.yaml        # NGINX Ingress ArgoCD 애플리케이션
├── monitoring-argocd.yaml     # 모니터링 스택 ArgoCD 애플리케이션
├── project-secret.yaml        # ArgoCD 프로젝트 관련 Secret (필요시 사용)
└── vllm-eval-argocd.yaml      # vLLM 평가 Job ArgoCD 애플리케이션
└── vllm-nextchat-argocd.yaml  # vLLM-NextChat ArgoCD 애플리케이션
```

## 배포 방법

이 디렉토리의 매니페스트들은 ArgoCD가 Git 리포지토리를 바라보도록 설정하여 적용하는 것이 가장 이상적입니다. 하지만 수동으로 각 애플리케이션을 배포할 수도 있습니다.

### 권장: ArgoCD App of Apps 패턴

1.  **ArgoCD 설치**: 클러스터에 ArgoCD가 설치되어 있어야 합니다.
2.  **리포지토리 등록**: 이 Git 리포지토리를 ArgoCD에 연결합니다.
3.  **App of Apps 배포**: 이 디렉토리 전체를 가리키는 하나의 "Root" ArgoCD 애플리케이션을 배포합니다. 이 Root 앱이 다른 모든 컴포넌트 앱(`monitoring-argocd.yaml`, `vllm-nextchat-argocd.yaml` 등)을 자동으로 배포하고 동기화합니다.

### 수동 배포

클러스터에 직접 각 컴포넌트를 배포하려면 `kubectl`을 사용합니다.

```bash
# 원하는 컴포넌트의 ArgoCD Application을 클러스터에 적용합니다.
# 예시: vllm-nextchat 배포
kubectl apply -f argocd-project/vllm-nextchat-argocd.yaml

# 예시: 모니터링 스택 배포
kubectl apply -f argocd-project/monitoring-argocd.yaml

# 모든 컴포넌트 한 번에 배포
kubectl apply -k argocd-project/
# 또는
kubectl apply -f argocd-project/
```

각 ArgoCD Application은 `spec.source.path`에 정의된 경로의 매니페스트를 참조하여 해당 컴포넌트를 클러스터의 지정된 네임스페이스(`spec.destination.namespace`)에 배포합니다. 