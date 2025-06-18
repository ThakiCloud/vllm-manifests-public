# vLLM-Eval 배포 가이드

## 개요
이 문서는 `vllm-eval/` 디렉토리의 리소스를 사용하여 vLLM 모델 성능 평가 시스템을 배포하고 실행하는 방법을 설명합니다. 이 시스템은 ArgoCD를 통해 GitOps 방식으로 관리되거나, 수동으로 Job을 실행할 수 있습니다.

## 포함된 매니페스트
- `vllm-eval-argocd-app.yaml`: `vllm-eval` 디렉토리의 리소스들을 관리하는 ArgoCD Application 정의입니다. GitOps 워크플로우를 사용할 경우에만 필요합니다.
- `vllm-benchmark-job.yaml`: vLLM 엔드포인트의 성능(throughput, latency)을 벤치마킹하는 Kubernetes Job입니다.
- `evalchemy-job.yaml`: 특정 설정 파일(`eval_config.json`)을 기반으로 모델의 품질을 평가하는 Kubernetes Job입니다.

## 배포 시나리오

이 컴포넌트는 두 가지 방식으로 배포할 수 있습니다.

### 시나리오 1: ArgoCD를 이용한 GitOps 배포 (권장)

이 방식은 `vllm-eval` 관련 매니페스트를 ArgoCD가 관리하도록 설정합니다.

1.  **ArgoCD에 리포지토리 등록**:
    만약 이 Git 리포지토리가 ArgoCD에 등록되어 있지 않다면 먼저 등록해야 합니다.

2.  **ArgoCD Application 배포**:
    아래 명령어로 `vllm-eval-argocd-app.yaml`을 클러스터에 배포합니다.

    ```bash
    kubectl apply -f vllm-eval/vllm-eval-argocd-app.yaml
    ```
    이후 ArgoCD UI에서 `vllm-eval-argocd-app` 애플리케이션이 생성되고 동기화되는 것을 확인할 수 있습니다. Job들은 자동으로 실행되지 않으며, ArgoCD UI에서 수동으로 Trigger하거나 스케줄에 따라 실행되도록 추가 설정이 필요할 수 있습니다.

### 시나리오 2: 수동으로 Job 실행

필요할 때마다 평가 Job을 직접 실행하는 방식입니다.

1.  **네임스페이스 생성**:
    평가 Job을 실행할 네임스페이스를 생성합니다. (예: `vllm-nextchat`)

    ```bash
    kubectl create namespace vllm-nextchat
    ```

2.  **Private Registry Secret 생성 (필요시)**:
    `ghcr.io` 등 Private 컨테이너 레지스트리에서 이미지를 받아오려면 `ghcr-secret`과 같은 이미지 풀 시크릿을 해당 네임스페이스에 생성해야 합니다.

3.  **Job 매니페스트 수정**:
    `vllm-benchmark-job.yaml`과 `evalchemy-job.yaml` 파일 내의 `env` 섹션에서 `VLLM_ENDPOINT` 등 평가 대상 환경에 맞는 값으로 수정합니다. 또한, 결과 저장을 위한 `PersistentVolumeClaim` 설정을 확인하고 환경에 맞게 수정합니다.

4.  **Job 실행**:
    수정된 Job 매니페스트를 `kubectl apply`를 사용하여 클러스터에 제출합니다.

    ```bash
    # 벤치마크 Job 실행
    kubectl apply -f vllm-eval/vllm-benchmark-job.yaml -n vllm-nextchat
    
    # Evalchemy Job 실행
    kubectl apply -f vllm-eval/evalchemy-job.yaml -n vllm-nextchat
    ```

## 배포 후 확인
Job 실행 후, 아래 명령어로 Pod의 상태를 확인하여 평가가 정상적으로 수행되고 있는지 모니터링할 수 있습니다.

```bash
# vllm-nextchat 네임스페이스의 Pod 목록 확인
kubectl get pods -n vllm-nextchat

# 특정 Job Pod의 로그 확인
kubectl logs <job-pod-name> -n vllm-nextchat -f
```
Job이 완료(`Completed`)되면 PVC에 마운트된 경로에서 결과 파일을 확인할 수 있습니다. 