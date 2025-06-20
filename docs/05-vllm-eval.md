# vllm-eval: 모델 성능 평가

`vllm-eval` 디렉토리는 배포된 vLLM 모델의 성능을 측정하기 위한 Kubernetes Job 매니페스트를 포함합니다. 부하 테스트를 통한 성능 벤치마킹과 특정 데이터셋을 이용한 정성/정량 평가를 수행할 수 있습니다.

## 디렉토리 구조

```
vllm-eval/
├── evalchemy-job.yaml      # Evalchemy를 이용한 정성/정량 평가 Job
└── vllm-benchmark-job.yaml # vLLM 엔드포인트 부하 테스트 Job
```

## 배포 및 실행 방법

평가 Job은 필요할 때마다 수동으로 실행하는 일회성 작업입니다.

1.  **네임스페이스 생성**:
    Job을 실행할 네임스페이스를 생성합니다. (예: `vllm-eval`)
    ```bash
    kubectl create namespace vllm-eval
    ```

2.  **Job 매니페스트 수정**:
    각 Job의 YAML 파일을 열어 **환경 변수**나 **args**를 수정해야 합니다.
    -   `vllm-benchmark-job.yaml`: `HOST`, `PORT`, `MODEL_NAME` 등 벤치마크 대상 vLLM 서버의 정보를 수정합니다.
    -   `evalchemy-job.yaml`: 평가에 사용할 데이터셋, 모델, 평가 설정(`eval_config.json`) 등의 경로를 수정합니다. 보통 ConfigMap이나 PVC를 통해 이 설정들을 파드에 마운트합니다.

3.  **Job 생성 및 실행**:
    `kubectl create` 또는 `kubectl apply`를 사용하여 Job을 클러스터에 생성합니다. Job은 생성 즉시 실행됩니다.

    ```bash
    # 벤치마크 Job 실행
    kubectl create -n vllm-eval -f vllm-eval/vllm-benchmark-job.yaml

    # Evalchemy Job 실행
    kubectl create -n vllm-eval -f vllm-eval/evalchemy-job.yaml
    ```
    *참고: `kubectl apply`는 Job을 업데이트할 수 있지만, Job의 특정 필드는 변경할 수 없으므로 기존 Job을 삭제(`kubectl delete job ...`)하고 다시 생성해야 할 수도 있습니다.*

4.  **결과 확인**:
    Job 실행이 완료된 후, 로그를 통해 결과를 확인합니다.

    ```bash
    # 실행된 파드 이름 확인
    kubectl get pods -n vllm-eval

    # 파드 로그 확인 (예: vllm-benchmark-job-xxxxx)
    kubectl logs -n vllm-eval -f <job-pod-name>
    ```

    Job이 PVC(PersistentVolumeClaim)에 결과 파일을 저장하도록 구성된 경우, 해당 PVC를 마운트하는 다른 파드를 실행하여 결과 파일을 확인할 수도 있습니다.

## 자동화

ArgoCD Application(`argocd-project/vllm-eval-argocd.yaml`)을 사용하여 Git 변경에 따라 자동으로 Job을 실행하거나, Argo Events, GitHub Actions 등과 연동하여 CI/CD 파이프라인의 일부로 평가를 자동화할 수 있습니다. 