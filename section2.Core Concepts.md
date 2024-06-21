## Cluster Architecture

#### Contains

  - Kubernetes Architecture
  - ETCD For Beginners
  - ETCD in Kubernetes
  - Kube-API Server
  - Controller Managers
  - Kube Scheduler
  - Kubelet
  - Kube Proxy

#### Kubernetes의 목적

  - Application을 자동화된 방식의 Container 형식으로 호스트하는 것
  - 이로 인해 요구에 따라 Application의 많은 인스턴스를 쉽게 배포할 수 있고, Application 내 다양한 서비스 간의 통신이 쉽게 가능

#### Kubernetes Cluster는 Node set으로 구성되는데 물리적, 가상(virtual), on-premise, 또는 cloud일 수 있고, container 형태의 Application host일 수 있음

#### Kubernetes Architecture (화물선에 비유)

  - 모든 Node에는 `Container Runtime Engine` 설치되어 있음
  - `Worker Node`는 container를 로딩할 수 있는 배

    - Host Application as Containers

    1. `kubelet` : cluster의 모든 worker node에서 실행되는 agent. Master node의 Kube API server의 지시를 듣고 container를 배포하거나 파괴. Master node의 Kube API server에 node와 container의 상태 보고서 제공
    2. `kube proxy` : Worker Node 간의 통신. Worker Node에 필요한 규칙이 시행되도록 함. 그 위에서 실행되는 container가 통신이 가능하도록 함

  - 선박에 container를 적재하고, 어떻게 적재할지 계획하고, 선박을 식별하고, 그에 대한 정보를 저장하고, 선박에 실린 container의 위치를 감시하고 추적하는 등의 전적인 적재 과정을 관리하는 관제선 역할의 `Master Node`
    - Manage, Plan, Schedule, Monitor Nodes
    - Master Node는 cluster를 관리하고, 서로 다른 노드에 대한 정보를 저장하고, 어떤 container가 어디로 갈지 계획하고, node와 container를 모니터링하는 역할
    1. `ETCD Cluster` : Cluster에 관한 정보 저장, key-value 형식으로 정보를 저장하는 DB
    2. `Kube Scheduler` : container를 설치하기 위한 올바른 node 식별, container resource 요구 사항이나 worker node의 용량, 다른 정책(policy)이나 제약 조건들, taint-toleration과 같은 규칙에 근거
    3. `Controller Manager` : Node controller가 node 관리(새 node를 cluster에 온보딩하고, 노드가 사용 불가능하거나 파괴되는 상황 처리), Replication controller가 replication 관리(원하는 container 수가 replication에서 항상 실행되도록 보장)
    4. `Kube API server` : Kubernetes 주요 관리 구성 요소. Cluster 내에서의 모든 작업을 오케스트레이션. 주기적으로 Worker Node의 Kubelet으로부터 상태 보고서를 가져와 node와 container 상태 모니터링

#### Container
- Application은 Container 형태
- Master Node에 전체 관리 시스템을 형성하는 다양한 구성 요소는 container 형태로 호스트될 수 있음
- DNS Service Networking Solution은 Container 형태로 배포될 수 있음
- 따라서 container를 실행할 소프트웨어 필요 => `Container Runtime Engine` (Ex. Docker, ContainerD, Rocket)
  - Container Runtime Engine은 cluster 내 모든 Node에 설치되어 있음 (Master node(control 구성 요소를 container로 host하고 싶은 경우), worker node)

## Docker vs ContainerD

1. Docker

- Container가 발명되기 이전 Docker와 Rocket 존재
- container 작업이 간단해져 가장 지배적인 container 도구가 됨
- Kubernetes가 Docker의 지휘를 맡음 (과거에 Kubernetes는 Docker만 담당하고 다른 container solution은 지원하지 않음)
- 이후 Kubernetes가 다른 container runtime 지원
- Kubernetes는 `CRI`(Container Runtime Interface) 소개

  - CRI는 Kubernetes와 다른 container runtime으로 작업하게 해줌
  - OCI(Open Container Initiative) 표준을 준수한다는 가정 하에 동작 - imagespec(image 빌드 방식)과 runtimespec으로 구성 - 누구나 kubernetes와 작업할 수 있는 container runtime 만들 수 있음
    > OCI 표준을 준수하는 모든 container runtime은 CRI를 통해 kubernetes의 container runtime 지원 가능

- **BUT Docker는 CRI 표준을 지원하려고 만든 것이 아님**

  - Docker는 CRI가 나오기 전에 만들어졌고 여전히 주요 Container 도구였기 때문에 Kubernetes는 Docker를 계속 지원해야 했음
  - 따라서 Docker를 지원하기 위해 `dockershim`을 임시 방편으로 도입 => CRI 밖에서 Docker를 지속적으로 지원하기 위한 목적

- **Docker는 container runtime 기능만 존재하는 것이 아님**

  - Docker 구성: CLI, API, BUILD, VOLUMES, AUTH, SECURITY, CONTAINERD(container runtime)
  - ContainerD는 CRI 호환이 가능하고 다른 runtime처럼 Kubernetes와 직접적으로 작업할 수 있음
  - 따라서 ContainerD는 Docker와 별도로 Runtime으로 사용 가능

- Kubernetes는 Docker engine(`ContainerD`) 직접 지원
  - dockershim을 유지하기 위한 불필요한 노력으로 문제가 커짐
  - 따라서 dockershim을 완전히 제거하기 위해 kubernetes v1.24를 발표하고 docker 지원 해제
  - Docker 이미지는 OCI 표준에서 imagespec을 만족하기 떄문에 Containerd와 계속 동작
    > Docker 전체가 아닌 ContainerD를 직접 연결함으로써 다른 Docker 기능은 더이상 지원하지 않음. BUT image는 OCI 표준을 만족하기에 ContainerD와 계속 동작

2. ContainerD

- Docker의 일부이지만 현재는 독립된 프로젝트
- Docker를 설치할 필요없이 ContainerD 자체 설치 가능
- 일반적으로 Docker를 사용해 명령어를 실행하는 container를 실행했는데, docker가 설치 안 되었다면 containerD만 장착된 container 실행하는 방법 => `ctr`

- **ctr**

  - ContainerD 설치 -> `ctr` command 사용 가능
  - Not very user friendly
  - Only supports limited features
    - 디버깅 container를 위해 만들어짐
  - 이미지 pull
    - `ctr images pull docker.io/library/redis:alpine`
  - container 실행
    - `ctr run docker.io/library/redis:alpine redis`

- **nerdctl**

  - ctr보다 더 나은 대안
  - Docker의 CLI와 아주 유사
  - Docker가 지원하는 대부분의 옵션 지원
  - containerd에 구현된 최신 기능에 액세스 가능
    - Encrypted container images
      - 암호화된 container image로 작업 가능
    - Lazy Pulling
      - Lazy한 이미지 pulling 가능
    - P2P image distribution
      - P2P 이미지 배포 가능
    - Image signing and verifying
    - Namespaces in Kubernetes
  - container 실행
    - `nerdctl run --name redis redis:alpine`
  - port 매핑
    - `nerdctl run --name webserver -p 80:80 -d nginx`

- **crictl**
  - Kubernetes의 CRI(container runtime interface)와 호환되는 container runtime과 상호작용하는 데 사용
    - 다양한 runtime에 걸쳐 작동
  - Installed separately
    - 별도로 설치해야 함
  - Used to inspect and debug container runtimes
    - Not to create containers ideally
    - 검사와 container runtime에 사용되고, ctr과 nerdctl과 달리 container 생성에 사용되지 않음 (디버깅 도구)
  - Works across different runtimes
  - Kubernetes의 kubelet과 잘 어울려 생성에는 사용되지 않고 감시에만 사용
    - kubelet이 한 번에 특정 container나 pod를 특정 개수만큼 node에서 사용할 수 있게 보장
    - crictl이 container를 생성하면 kubelet이 삭제 (kubelet은 지정된 개수만큼만 존재하도록 보장)
  - `crictl pull busybox`
  - `crictl images`
  - `crictl ps -a`
  - container 내 명령 실행
    - `crictl exec -i -t [CONTAINER ID] ls`
  - log
    - `crictl logs [CONTAINER ID]`
  - pod
    - `crictl pods`
    - crictl은 docker와 달리 pod 인식 가능

> dockershim은 cri-dockerd로 변화

## ETCD For Beginners

> ETCD is a distributed reliable key-value store that is Simple, Secure & Fast

### key-value store

1. Tabular/Relational Databases
- 행과 열의 형태로 데이터 저장
- 새로운 정보가 추가될 때마다 테이블 전체가 영향을 받아 빈 공간이 많아짐

2. key-value store
- 정보를 문서나 페이지 형태로 보관
- 하나의 문서에 해당하는 모든 정보 저장
- 다른 문서를 업데이트할 필요 없이 하나의 문서만 업데이트 가능
- 데이터가 복잡해지면 JSON이나 YAML 타입의 데이터 포맷을 사용할 수 있음

### Install ETCD

1. Download Binaries

`curl -L [BINARY ADDRESS] -o [저장할 파일명]`

2. Extract

`tar xzvf [파일명]`

3. Run ETCD Services

`./etcd`

### Operate ETCD

> 직전에 ETCD 실행 명령어를 실행하면 2379 port가 열림

- 2379 port를 통해 정보를 저장하고 검색할 수 있음
- etcd 설치 시 `etcdctl` 명령어를 사용할 수 있게 됨
  - etcdctl은 etcd의 command line client
  - key-value를 저장하고 회수할 수 있음
  - 저장
    - `./etcdctl set key1 value1`
  - 회수
    - `./etcdctl get key1`
  - 명령어 조회
    - `./etcdctl`

### ETCD Versions

1. v0.1 (2013)
2. v0.5 (2014)
3. v2.0 (2015)
- 2015에 RAFT 합의 알고리즘 재설계
4. v3.1 (2017)
- 최적화와 성능 향상이 훨씬 더 많이 이루어짐
5. CNCF Incubation (2018)

> 이때 version2.0과 version3.1 사이의 변화를 잘 봐야 함

- 위에서 진행한 etcdctl 명령은 API v2.0과 작동하도록 구성

- `./etcdctl --version` 결과는 etcdctl version과 API version 두 개 존재
- API version 2와 3는 etcdctl 명령어에 차이가 있으므로 구분해야 함

- API Version 변경
  - `ETCDCTL_API=3 ./etcdctl version` 또는 `export ETCDCTL_API=3 ./etcdctl version`
  - 전자는 일회성으로 환경 변수 설정, 후자는 영구적으로 이후에 실행되는 모든 명령어에 영향 미침

### ETCDCTL v3

- 명령어 조회
  - `./etcdctl`
- 저장
  - `./etcdctl put key1 value1`
- 회수
  - `./etcdctl get key1`

## ETCD in Kubernetes

> kubectl get 명령어 사용 시에는 etcd DB로부터 데이터를 가져옴

> Kubernetes 배포(cluster 설정)에는 Scratch와 Kubeadm Tool 두 가지 방식이 있음

### Setup - Manual

- Cluster를 Scratch에서 셋업하는 경우 바이너리를 직접 다운로드해 배포
- Master node에 직접 바이너리를 설치하고 서비스로서 구성
- 대부분이 certificate(ex.TLS)와 관련
- 유일한 옵션이 `--advertise-client-urls https://${INTERNAL_IP}:2379`
  - etcd가 듣는 주소
  - server의 IP와 port 2379
  - kube api server에서 구성되어야 할 URL

`wget -q --https-only \"https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"`

![alt text](image.png)

### Setup - kubeadm

> kubeadm이란 쿠버네티스 클러스터를 빠르고 쉽게 구축하기 위해 제공하는 도구

- kubeadm이 kube system namespace에 있는 pod로 서버 배포

`kubectl get pods -n kube-system`
![alt text](image-1.png)

- etcdctl로 DB에 접근 가능
  - Kubernetes가 저장한 모든 key를 열거하려면 아래와 같이 명령어 실행
  - `kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only`
  - Kubernetes는 특정 디렉토리 구조에 데이터 저장
    - Root registry는 registry로, 그 아래에 minions, pods, replications과 같은 다양한 쿠버네티스 리소스 존재
  ![alt text](image-2.png)

### ETCD in HA Environment

- 고가용성 환경(High Availability)에선 클러스터에 Master Node가 여러개
- Master Node 전체에 여러 개의 etcd instance가 퍼지고 이 경우 etcd 서비스 구성에 올바른 매개 변수를 설정해 etcd instance가 서로 알도록 설정
 
![alt text](image-3.png)


## ETCD - Commands

## Kube-API Server

## Kube Controller Manager

## Kube Scheduler

## Kubelet

## Kube Proxy
