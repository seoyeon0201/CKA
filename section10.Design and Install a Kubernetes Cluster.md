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

## ETCD in HA
