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


## Configure High Availability

## ETCD in HA
