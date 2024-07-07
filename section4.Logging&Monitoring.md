## Monitor Cluster Components

#### Monitor

- Kubernetes의 자원 소비 모니터링하는 방법
    - Node level: Cluster 내 Node 개수, Health, CPU와 memory, Network disk 활용도
    - Pod level: pod 개수, 각 pod 성능, CPU와 memory 사용량

- 이 metric을 모니터링하고 저장하고 데이터에 대한 분석을 제공할 수 있는 솔루션 필요 => Monitoring Solution
    - Kubernetes에는 풀 기능이 탑재된 모니터링 솔루션 없음
    - 다양한 오픈소스 솔루션 존재
        - Metrics Server, Prometheus, Elastic Stack, Datadog, dynatrace 등

#### Heapster vs Metrics Server

1. Heapster
- Kubernetes의 모니터링과 분석을 가능하게 한 초기 프로젝트 중 하나
- Kubernetes 모니터링에서 참조 아키텍처를 찾으면 온라인에서 많은 참조를 찾을 수 있음
- Deprecated

2. Metrics Server
- Heapster가 Deprecated되고 Metrics Server라는 간소화된 버전 등장

#### Metrics Server

- Cluster당 Metric Server 1개
- Kubernetes Node와 pod에서 metric 회수해 메모리에 저장
    - In-memory 모니터링 솔루션으로, 디스크에 metric을 저장하지 않음
- Pod에 관련된 metric은 각 node의 kubelet 실행
    - kubelet은 cAdvisor를 하위 요소로 포함하고 있음
    - cAdvisor는 pod에서 성능 metric을 회수하고 kubelet API를 통해 metric을 공개해 metric server에서 metric이 사용 가능하도록 함

#### Metrics Server - Getting Started

`minikube addons enable metrics-server`

- Local cluster에서 minikube 사용한다면 addons가 metric server를 활성화하도록 함

`git clone https://github.com/kubernetes-incubator/metrics-server.git`

`kubectl create -f deploy/1.8+/`

- 그 외의 경우, 깃허브에서 metrics server 파일 복제해 metrics server 배포

#### View

`kubectl top node`

- 각 node의 성능 지표(CPU, memory 소비) 제공

`kubectl top pod`

- 각 pod의 성능 지표 확인

## Practice Test - Monitor Cluster Components

| Metrics server 배포 방법

1. 깃에서 metrics-server 배포 파일 클론

`git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git`

2. 해당 파일 들어가서 Metric server 배포

`cd kubernetes-metrics-server`
`kubectl create -f .`

| Metrics server 사용

`kubectl top node`

- node의 리소스 사용량 조회

`kubectl top pods`

- pod의 리소스 사용량 조회

## Managing Application Logs

#### Log - Docker

`docker run kodekloud/event-simulator`

- 로그 조회

`docker run -d kodekloud/event-simulator`

- log 보이지 않음

`docker logs -f ecf`

- 실시간 log 조회

#### Log - Kubernetes

`kubectl logs -f [POD NAME]`

- -f 옵션으로 실시간 로그 조회 가능
- 이 log는 container 로그

`kubectl logs -f [POD NAME] [CONTAINER NAME]`

- pod는 여러 개의 container를 담을 수 있는데, 이 경우 container의 로그를 조회하려면 명령어에 container 이름을 명시해야 함

## Practice Test - Monitor Application Logs

`kubectl logs -f [POD NAME]`

- 실시간으로 pod의 container의 로그 조회 가능

`kubectl logs -f [POD NAME] [CONTAINER NAME]`

- 실시간으로 해당 container 로그 조회 가능