## Design and Install a Kubernetes Cluster

#### Ask

| cluster를 설계하기 전 아래의 질문 진행

- Purpose
    - Education
    - Development & Testing
    - Hosting Production Applications

- Cloud or OnPrem?

- Workloads?
    - How many?
    - What kind?
        - Web
        - Big Data/Analytics
    - Application Resource Requirements
        - CPU Intensive
        - Memory Intensive
    - Traffic
        - Heavy traffic
        - Burst traffic

#### Purpose

| Cluster 배포 목적

1. Education

- Minikube
- Single node cluster with kubeadm/GCP/AWS

2. Development & Testing

- Multi-node cluster with a Single Master and Multiple workers
- Setup using kubeadm tool or quick provision on GCP or AWS or AKS

3. Hosting Production Applications

- High Availability Multi node cluster with multiple master nodes
    - Production application hosting에는 다수의 master node가 있는 고가용 multi-node cluster 추진
- Kubeadm or GCP or Kops on AWS or other supported platforms
- Upto 5000 nodes
- Upto 150,000 PODs in the cluster
- Upto 300,000 Total Containers
- Upto 100 PODs per Node

=> 이 세션에서 Multi master node를 이용한 high availability setup에 관해 좀 더 살펴볼 것

- Cluster 크기에 따라 node의 리소스 요구 사항은 다양함
    - GCP 또는 AWS 같은 Cloud Service Provider는 cluster 내 node의 수에 근거해 자동으로 적당한 크기의 node 선택
    - 아래 표는 특정 node 개수에 따른 instance 크기와 리소스 사양을 나타냄

|Nodes|GCP||AWS||
|:--:|:--:|:--:|:--:|:--:|
|1-5|N1-standard-1|1vCPU 3.75GB|M3.medium|1vCPU 3.75GB|
|6-10|N1-standard-2|2vCPU 7.5GB|M3.large|2vCPU 7.5GB|
|11-100|N1-standard-4|4vCPU 15GB|M3.xlarge|4vCPU 15GB|
|101-250|N1-standard-8|8vCPU 30GB|M3.2xlarge|8vCPU 30GB|
|251-500|N1-standard-16|16vCPU 30GB|C4.4xlarge|16vCPU 30GB|
|>500|N1-standard-32|32vCPU 120GB|C4.8xlarge|32vCPU 60GB|

#### Cloud or OnPrem?

| Cloud든 On-Premise든 이미 모든 배포 옵션은 모든 환경에서 가능

1. Use Kubeadm for on-prem 

- On-premise 환경에서 Kubeadm은 유용한 도구

2. GKE for GCP

- Google Container Engine은 GCP에서 Kubernetes cluster provisioning을 아주 쉽게 해줌
- cluster 유지를 쉽게 해줌
    - 클릭 한 번에 cluster 업그레이드 기능도 가능


3. Kops for AWS

- Kops는 AWS에서 Kubernetes cluster를 배포하는 좋은 도구

4. Azure Kubernetes Service(AKS) for Azure

- AKS는 Azure에 호스트된 kubernetes 환경을 관리하는 것을 도움

#### Storage

| 구성된 작업에 따라 node와 configuration이 다름

- High Performance - SSD Backend Storage
    - 고성능 작업은 SSD 백업 저장소에 의존
- Multiple Concurrent connections - Network based storage
    - 다중 동시 액세스는 Network based storage 고려
- Persistent shared volumes for shared access across multiple PODs
    - 다중 pod의 volume shared 액세스를 위해서는 Persistent shared volumes 사용 
- Label nodes with specific disk types
- Use Node Selectors to assign applications to nodes with specific disk types

#### Nodes

- Virtual or Physical Machines
    - Kubernetes cluster에서 형성되는 node는 물리적일 수도 가상일 수도 있음
    - 우리는 cluster의 node로 가상 컴퓨터 환경을 Virtual Box로 배포
    - 물리적 컴퓨터나 가상 컴퓨터 또는 GCP, AWS, Azure 환경에 배포하는 것을 선택할 수 있음
- Minimum of 4 Node Cluster (Size based on workload)
- Master vs Worker Nodes
    - Master node는 구성 요소를 호스팅하는 데 사용
        - Ex. kube-apiserver, etcd server
    - Worker node는 작업 호스팅
- Linux x86_64 Architecture
    - node로 64bit Linux OS를 반드시 사용해야 함

- Master nodes can host workloads
    - master node도 node로 간주되어 작업을 호스팅할 수 있음
- Best practice is to not host workloads on Master nodes
    - BUT Master node를 오직 component를 컨트롤하기 위해 지정하는 것 추천
    - kubeadm과 같은 배포 도구는 master node에 taint를 추가해 작업이 master node에서 호스팅되는 것 방지

#### Master Nodes

- 일반적으로 Controlplane의 모든 구성 요소를 master node에 배치
- 하지만 큰 cluster에서 ETCD cluster를 자체 cluster node로 분리할 수 있음


## Choosing Kubernetes Infrastructure

| Kubernetes는 다양한 시스템에 다양한 방식으로 배포: Labtop(=local device),Physical server, Cloud server

#### Labtop

1. 지원되는 Linux 컴퓨터에 수동으로 binary 설치해 local cluster 설정

2. 위 방법을 자동화하는 솔루션에 의존해 cluster 설정

3. Windows에서는 Kubernetes를 원래 설정으로 설정할 수 없음

- 바이너리가 없기 때문
- Hyper-V 또는 VMware Workstation, VirtualBox와 같은 가상 소프트웨어에 의존해 Linux VM 생성해 Kubernetes 환경 설정 가능

| Kubernetes 구성 요소를 실행하는 솔루션

- Windows VM Docker container로서의 `Docker`
- Docker 이미지는 Linux 기반으로 그 아래에는 HyperV에 의해 Linux docker container를 실행하도록 만든 작은 Linux OS 실행

| Local computer에서 쉽게 Kubernetes를 시작하는 솔루션

- `Laptop`
    - 위에 진행

- `minikube`
    - 단일 node cluster를 쉽게 배포
    - Oracle VirtualBox와 같은 가상화 소프트웨어에 의존해 Kubernetes cluster 구성 요소를 실행하는 가상 컴퓨터 생성

- `kubeadm`
    - 하나의 node 또는 다중 node cluster를 빠르게 배포
    - 이를 위해 지원되는 구성과 함께 필요한 host를 직접 프로비전해야함

- minikube vs kubeadm
    - minikube는 VM을 배포하고 kubeadm은 VM은 이미 프로비전(준비)되었다고 기대
    - minikube는 single node cluster 배포하고, kubeadm은 single 또는 multi node cluster 배포

- Kubernetes cluster를 노트북에 local로 배포하는 것은 보통 학습, 테스트, 개발 목적

#### Kubernete cluster 시작 방법

| Kubernetes cluster를 시작하는 방법은 많음 => Private 또는 Public Cloud 환경

1. Turnkey Solutions

- You Provision VMs
    - 필요한 VM을 프로비전하는 곳
- You Configure VMs
- You Use Scripts to Deploy Cluster
    - 도구나 스크립트를 이용해 kubernetes cluster 구성
- You Maintain VMs yourself
    - VM을 관리하고 패치하고 업그레이드하는 작업 할 수 있음
- Eg: Kubernetes on AWS using KOPS
    - cluster 관리와 유지는 이 도구와 스크립트를 사용하면 쉬워짐
    - KOPS를 이용해 AWS에 Kubernetes cluster 배포

2. Hosted Solutions (=Managed Solutions)

- Kubernetes-As-A-Service
    - Service 솔루션에서 Kubernetes와 비슷
- Provider provisions VMs
    - Cluster와 요구되는 VM이 공급자에 의해 배포되고 구성
- Provider installs Kubernetes
- Provider maintains VMs
    - VM은 공급자에 의해 유지됨
- Eg: Google Container Engine (GKE)

#### Turnkey Solutions

1. OpenShift

- Redhat의 온프레미스 Kubernetes 플랫폼
- 오픈 소스 container application platform으로 Kubernetes에 기반하여 만들어짐
- 추가적인 도구 집합과 Kubernetes의 구성물을 생성하고 관리할 수 있는 좋은 GUI 제공
    - CI/CD 파이프라인 등과 쉽게 통합할 수 있도록 함
- 카탈로그에 초급반 Openshift 존재

2. Cloud Foundry Container Runtime

- 오픈 소스 도구를 이용해 고가용성 Kubernetes Cluster 배포 및 관리를 도움

3. VMware Cloud PKS
- Kubernetes를 위한 VMware 환경을 활용하고 싶다면 VMware Cloud PKS 솔루션 사용

4. Vagrant

- 유용한 스크립트를 제공해 서로 다른 cloud service provider에 kubernetes cluster 배포

| 위 솔루션을 통해 Kubernetes cluster를 배포하고 개별적으로 관리하는 것을 쉽게 해줌. 단 지원되는 구성 설정이 포함된 가상 컴퓨터가 존재해야하며, Kubernetes Certified Solution 중 일부

#### Hosted Solutions

1. GKE (Google Container Engine)

2. OpenShift Online
- RedHat의 제품으로 온라인에서 완전히 작동하는 Kubernetes cluster에 접속할 수 있음

3. Azure Kubernetes Service

4. Amazon Elastic Container Service for Kubernetes (EKS)

#### Our choice

- 학습 목적으로 하는 것이기 때문에 Public Cloud 계정에 접근하지 못 할 수 있음을 고려하면 Virtual Box로 Local 환경 선호
- Local system에 Local Kubernete cluster를 배포할 것
    - VirtualBox에 가상 컴퓨터 여러 개 생성

#### Our design

- 3개의 node
    - Master 1 Worker 2
- 가상 컴퓨터에 프로비전된 노트북에 배포될 것

## Configure High Availability

- Cluster의 Master node를 잃어버린다면?
    - Worker node가 작동하고 Container가 살아 있는 한 application은 작동
        - 사용자는 fail될 때까지 application에 접속 가능
- Container 또는 Pod가 고장난다면?
    - Pod가 ReplicaSet의 일부라면 Master의 replication controller가 worker node에게 새 pod를 생성하라고 지시
    - BUT Master node를 사용할 수 없음
        - Master node의 controller와 scheduler 사용할 수 없음
        - Kube-apiserver도 사용할 수 없어 kubectl 툴이나 API를 통해 외부적으로 cluster에 액세스할 수 없음

=> 따라서 Production 환경에서 고가용성 구성으로 Multiple Master node를 고려해야 함

- High Availability Configuration
    - Cluster 내 모든 구성 요소를 중복으로 가지는 것
        - 단일 실패 지점을 피하기 위함
    - Master node, Worker node, Controlplane component, Application이 Replicaset 및 Service 형식으로 여러 개의 복사본 존재

| Master node의 Controlplane component를 살펴볼 것

#### Master node

| 지금까지 Cluster 내의 3개의 Node(1 Master node, 2 Worker node) 존재 => Master node만 집중적으로 살펴볼 것 

- Master node는 controlplane 구성 요소 호스팅
    - kube-apiserver, controller scheduler, etcd server 

- 추가적인 Master node와 함께 설치하면 새로운 master node에도 동일한 구성 요소 실행
- 2개의 Master node 동작
    - 같은 component의 여러 instance를 실행하면 같은 일을 2번 하는가? or 일을 어떻게 분담하는가?
        - 무엇을 하냐에 따라 다름

1. `kube-apiserver`
- 요청을 수신하고 프로세싱하고 cluster에 관한 정보 제공
- 요청에 따라 한 번에 하나씩 작업되어 cluster node 전체의 apiserver가 active 모드가 되어 동시에 실행
- 지금까지는 kubectl 유틸리티와 apiserver가 교신해 작업 완료
- 그 kubectl 유틸리티를 master node1의 port 6443으로 가도록 함 => `https://master1:6443`
    - 해당 port는 apiserver가 듣는 곳
    - kubeconfig 파일에 구성되어 있음
- master node가 2개인데 kubectl 2는 어디로 가는가? => `https://master2:6443`
    - master node의 어느 쪽이든 요청할 수 있지만 양쪽 다 같은 요청을 해서는 안 됨
- 따라서 일종의 `load-balancer`를 가지는게 좋음 => `https://load-balancer:6443`
    - Master node의 앞에서 apiserver 간의 트래픽 분할 
    - 그런 다음 kubectl 유틸리티가 load balancer를 가리키도록 함
    - 이 목적으로 nginx, HAProxy 또는 그 외의 다른 load balancer 사용 가능

2. `controller manager`
- Cluster 상태를 보고 행동을 취함
- Controller로 구성되어 있는데, replication controller처럼 pod 상태를 계속 관찰하고 pod 하나가 고장나면 새 pod를 만드는 등 필요한 조치 취함
- 다수의 instance가 병렬로 실행되면 작업을 중복해 필요 이상으로 pod를 늘릴 수 있음
- 따라서 병렬로 실행되면 안 됨 => `Active & Standby 모드`
    - 하나의 Master node의 controller manager는 Active 모드, 다른 master node의 controller manager는 Standby 모드
    - `주도적인 선거 과정`을 통해 Active, Standby 모드 부여
        - Controller Manager 프로세스가 구성되면 `leader elect` 옵션 지정 가능
            - `kube-controller-manager --leader-elect true [other options]`
            - default가 true 
        - Controller Manager 프로세스가 시작되면 `Kube-controller-manager Endpoint`라는 이름의 endpoint 개체가 lease 또는 lock을 얻음
        - 어떤 프로세스든 해당 정보와 함께 endpoint를 업데이트하는 쪽이 lease를 획득해 두 master node 중 하나가 활성화되고 다른 하나는 비활성화
        - 기본 duration은 15초. 해당 시간동안 지정된 lease를 위해 다른 node에 lock 설정
            - `--leader-elect-lease-duration`
        - Active 프로세스가 lease 갱신. default 10s
            - `--leader-elect-renew-deadline 10s`
        - 위의 두 과정 모두 리더 선출이 정한 2초마다 leader가 되려고 노력함
            - `--leader-elect-retry-period 2s`
            - 그래야 첫 번째 master가 실패해도 두 번째 master가 로그를 획득해 리더가 될 수 있음

3. `scheduler`
- controller manager와 동일
- 병렬로 실행되면 안 됨 => `Active & Standby 모드` 실행
- 과정 모두 동일
    - command line 옵션 동일

4. `ETCD server`

- ETCD는 2가지 구성이 있음
    - 하나는 Kubernetes Master node의 일부로 ETCD 존재 => `Stacked Topology`
        - Easier to setup, manage, Fewer Servers
            - 설정과 관리가 쉽고 node도 더 적게 요구
        - Risk during failures
            - BUT 하나의 node가 죽으면 ETCD 둘 다를 포함한 controlplane instance 분실
    - 하나는 Controlplane Node에서 분리되어 그 자체 서버 세트에서 실행 => `External ETCD Topology`
        - Less Risky
            - 이전 topology와 비교해 이것은 덜 위험
            - controlplane node가 실패해도 etcd cluster와 저장된 데이터에 영향을 주지 않음
        - Harder to Setup
            - 설정이 어려움
        - More Servers
            - 외부 node에 대한 더 많은 서버(2배) 필요 

- API server는 ETCD server에 통신하는 유일한 구성요소
    - Topology에 상관없이 ETCD server 구성 장소가 동일한 server(node)이든 분리된 server이든 궁극적으로 API server가 해당 서버의 올바른 주소를 가리키도록 해야 함
    - API service 구성 옵션을 보면 etcd server 위치를 지정하는 옵션 존재
    - `cat /etc/systemd/system/kube-apiserver.service`에 `--etcd-servers=https://10.240.0.10:2379,https://10.240.0.11:2379`

=> `ETCD는 Distributed system`임을 기억할 것! API server나 통신하고자 하는 다른 구성 요소는 그 instance에서 해당 ETCD server에 도달할 수 있으며, ETCD server를 사용할 수 있는 instance를 통해 데이터를 읽고 쓸 수 있음

#### Our Design

- 단일 Master node에 2개의 Worker node로 구성했었지만, 현재 2개의 Master Node로 변경
- kube-apiserver에 대한 load balancer 장치 존재
- 총 5개의 Node
    - 2개의 Master node, 2개의 Worker node, 1개의 Load balancer

## ETCD in HA

- ETCD in High Availability 
- Cluster Configuration에 초점

#### Objectives

- What is ETCD?
- What is a Key-Value Store?
- How to get started quickly?
- How to operate ETCD?
- What is a distributed system?
- How ETCD operates?
- RAFT Protocol
- Best practices on number of nodes

#### ETCD

- ETCD is a distributed reliable key-value store that is Simple, Secure & Fast
    - ETCD는 분산되고 신뢰할 수 있는 key-value 저장소로, 간단하고 안전하고 빠름

#### Key-Value Store

- 전통적으로 데이터를 아래와 같은 테이블에 저장
    - Ex. 개인 정보 

|Name|Age|Location|Salary|Grade|
|:--:|:--:|:--:|:--:|:--:|
|John Doe|45|New York|5000||
|Dave Smith|34|New York|4000||
|Aryan Kumar|10|New York||A|
|Lauren Rob|13|Bangalore||C|
|Lily Oliver|15|Bangalore||B|

- key-value store는 문서나 페이지의 형태로 정보 저장
- 각 개인은 문서를 하나 씩 가지고 그 개인에 관한 모든 정보가 해당 파일에 저장됨
- 이 파일은 어떤 형식이나 구조로든 존재할 수 있음
- 한 파일의 변화는 다른 파일에 영향을 주지 않음
- 단순한 key와 value를 저장하고 회수할 수 있는 반면 데이터가 복잡해지면 JSON 또는 YAML 같은 데이터 포맷 사용 => `ETCD`

#### distributed

- 단일 서버에 추가했지만 database라 중요한 데이터를 저장할 수 있음
- 여러 서버에 걸쳐 데이터 저장소를 가질 수 있음
- 3개의 서버가 실행 중이면 데이터베이스의 동일한 복사본 유지
    - 하나를 잃어도 두개가 남음
- BUT 모든 노드의 데이터가 일관적인지 확인하는가? `Consistent`
    - 어떤 instance에서든 쓰고 데이터를 읽을 수 있음
    - ETCD는 데이터의 동일한 복사본이 모든 instance에서 동시에 사용 가능하도록 보장
    - `Read`: 같은 데이터가 모든 node에서 사용 가능하기 때문에 어떤 node에서든 쉽게 읽을 수 있음
    - `Write`: 2개의 write 요청이 2개의 다른 instance에서 오는 경우 
        - 두 node에 2개의 다른 데이터를 둘 수 없음
        - ETCD는 어떤 instance라도 쓸 수 있다고 했는데 100% 보장하지는 않음
        - ETCD는 node마다 write를 처리하지 않고 오직 한 instance에서 write 처리
        - 전체 instance에서 node 하나는 리더가 되고 다른 node는 추종자가 됨 => `Leader Election`
        - 리더 node를 통해 write 요청이 들어오면 리더 node가 write 진행 -> 리더 node가 다른 node에게 데이터 복사본 전송
            - 팔로워 노드를 통해 write 하면 팔로워 노드가 리더에게 write 전달
        - 따라서 다른 멤버들로부터 리더가 동의를 얻어야만 완성된 것으로 간주

#### Leader Election - RAFT

- 전체 instance에서 node 하나가 리더가 되고 나머지는 팔로워
- Leader Election 과정
    1. (팔로워 node를 통해 write하는 경우) 리더 node에게 write 요청 전달
    2. 리더 node가 write 진행
    3. 리더 node가 다른 node에게 데이터 복사본 전송

- 리더 node 선정 방법 - `RAFT Protocol` 사용

    | 3개의 master node 존재한다고 가정. Because ETCD는 Master node에 존재

    - 클러스터가 설정될 때 리더 node 선출되지 않음
    1. RAFT 알고리즘은 무작위로 타이머를 맞춰 신호 전송
    2. 타이머를 먼저 완료하는 사람이 다른 node에 리더 권한 요청
    3. 다른 node가 투표로 요청에 답변하면 리더 역할 맡게 됨
    4. 리더 node가 다른 master node에게 주기적으로 공지를 보내 자신이 리더 역할을 하고 있다고 알림
        - write 요청이 들어오면 리더 node에 의해 처리
        - cluster 내 다른 node로 복제
        - write는 cluster의 다른 instance로 복제되었을 때 완료되었다고 판단
        - ETCD는 High Availability 성질을 가져 node를 잃어도 기능함
            1) Majority 이상의 node에 write 가능하면 그 외의 node에 작성되지 않아도(노드 다운) 완료되었다고 간주
            - Majority = N/2 + 1
            - 단 1과 2의 Majority는 각각 1과 2
            - 즉 2개의 node의 경우 하나라도 down되면 write 완료되지 못함
            - 따라서 추가적인 cluster에 최소 3개의 instance를 갖도록 권장
            - 아래 표의 Quorum = Majority, Fault Tolerance는 작동하는 동안 손실할 수 있는 node 개수

                |Instances|Quorum|Fault Tolerance|
                |:--:|:--:|:--:|
                |1|1|0|
                |2|2|0|
                |3|2|1|
                |4|3|1|
                |5|3|2|
                |6|4|2|
                |7|4|3

            - Master node의 수 결정
                - 위 표에서 강조했듯이 **홀수개 선택**
                - 짝수 선택하는 경우 네트워크 세분화 중 cluster 결함이 발생할 가능성 있음
                - Ex. 6개의 node가 2개의 네트워크 망으로 나뉘는 경우 특히 3:3으로 나뉘는 경우 어떤 네트워크 망도 Quorum에 해당하는 4개를 active할 수 없음 
                - 홀수는 네트워크 세분화해도 cluster가 생존할 확률이 큼 
                - Ex. 7개의 node의 경우 2개의 네트워크 망으로 나뉘면 3:4로 Quorum에 해당하는 4개의 node가 active될 수 있음
                - BUT node가 5개 이상은 결함이 있을 위험이 커 X

            2) 다운된 node가 다시 올라오면 해당 node에도 데이터 복사

    5. 다른 node들이 리더 node로부터 어떤 시점에 대한 알림을 받지 못할 수 있음 => node 사이에서 재선출(re-election) 프로세스 시작 => 새 리더 선출
        - 리더 node 다운
        - 네트워크 연결 끊김

#### Getting Started

| 서버에 설치 방법 

1. 최신 지원되는 바이너리 다운로드
2. 다운로드한 바이너리를 압축해 필수 디렉토리 구조를 생성
3. 추가로 생성된 인증서 파일 위로 복사

```
wget -q --https-only "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"

tar -xvf etcd-v3.3.9-linux-amd64.tar.gz

mv etcd-v3.3.9-linux-amd64/etcd*/usr/local/bin/

mkdir -p /etc/etcd /var/lib/etcd

cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

4. ETCD Service 구성

- `etcd.service`
- 위 파일 내에서 중요한 것은 peer 정보를 전달하는 초기 cluster 옵션
    - 각 별도의 service가 해당 옵션을 통해 cluster의 일부와 peer가 어디 있는지 알 수 있음

`--initial-cluster peer-1=https://${PEER1_IP}:2380,peer-2=https://${PEER2_IP}:2380` 

#### ETCDCTL

| 설치 및 환경 설정이 끝나면 ETCDCTL 유틸리티를 이용해 데이터를 저장 및 검색

- Version2와 Version3 존재
    - 두 버전의 명령이 다름
    - Version2가 Default지만 Version3 사용할 것

```
export ETCDCTL_API=3    #ETCDCTL API Version 3로 설정. 이후의 etcdctl은 모두 version3로 진행

etcdctl put name john   #데이터 넣기. key가 name, value가 john

etcdctl get name        #key로 데이터 검색

etcdctl get / --prefix --keys-only  #ETCD에 존재하는 모든 key 찾기

```

#### Number of Nodes

- High Availability에 필요한 최소 master node는 3개
    - Majority(=Quorum) 관련
- Master node 최소 개수가 3개, 내결함성이 높은 것을 원한다면 5개 (그 이후로는 필요 X)
    - 자신의 환경과 결함 내구성, 요구 사항, 비용을 고려해 위에서 선택
    - 우리는 3개 선택

- 이전 섹션 Node
    -  Load Balancer 1개, Master node 2개, Worker node 2개

- 현제 섹션 Node
    - Load Balancer 1개, Master node 3개, Worker node 2개
        - BUT 노트북 한계로 Master node 2개만 사용. 다른 환경에서 셋업 배포하고 충분한 용량을 확보한다면 3개 사용
    - Stacked Topology 형태의 ETCD Server 사용
        - Master node에 ETCD가 내장되어 있음
