## Introduction to Deployment with Kubeadm

- Kubeadm은 multi node cluster 설정을 도움

#### kubeadm

- Kubernetes cluster는 다양한 구성 요소로 되어 있음
    - kube-apiserver, etcd, controller manager, scheduler, kubelet

- 모든 구성 요소 사이의 통신을 가능하게 하기 위한 보안과 인증에 대한 요구 사항 존재
- 다양한 구성 요소들을 다양한 node에 각각 설치하고 필요한 구성 파일 수정
    - 모든 구성 요소들이 서로를 가리키도록 인증서 설정 

=> kubeadm은 이 모든 과정 처리를 도움

#### Steps

| kubernetes cluster 셋업 단계

1. Multi System(=Node) 또는 VM Provisioning 필요

- VM Provision이 완료되면 가상 머신이 사용자의 요구 사항에 맞게 준비되어 실행됨
- Physical일 수 있고 Virtual일 수 있음

2. 하나는 Master node로, 나머지는 Worker node로 지정

3. Host(=Node)에 Container Runtime 설치

- 모든 node에 containerd 설치

4. 모든 node에 kubeadm 설치

- Kubernetes Solution 부트스트랩하도록 도움
- 필수 구성 요소를 올바른 순서대로 올바른 node에 설치

5. Master Server 초기화

- 이 프로세스 동안 필요한 구성 요소는 모두 Master Server에 설치 및 구성되어 있음

6. Pod Network 설정

- 네트워크 조건에 만족하는지 확인
- 일반적인 네트워크 연결은 충분하지 않음
- Pod Network 필요
    - Master node와 Worker node 사이

7. Worker node를 Cluster에 조인

- Worker node가 Master node와 Join됨
- Join Node


## Deploy with Kubeadm - Provision VMs with Vagrant

- Kubernetes Cluster를 위해 VM을 Provision하는 방법: Master node 1, Worker node 2 
- VirtualBox, Vagrant 소프트웨어 사용

- `VirtualBox`

    - Hypervisor
    - 궁극적으로 Virtual Machine을 실행하는 데 책임이 있음

- `Vagrant`

    - 자동화 도구
    - 특정 구성으로 VM을 쉽게 스핀업할 수 있도록 해줌
    - 정확히 동일한 설정을 따라 VM 실행
        - 단일 명령 하나로 VM 띄워 실행

| VirtualBox와 Vagrant 설치

#### VirtualBox

- 설치
    - `virtualbox.org` 좌측 Downloads에서 OS 선택

#### Vagrant

- 설치
    - `https://developer.hashicorp.com/vagrant/install?product_intent=vagrant`

- `Vagrant file`
    - VM을 위한 모든 구성 요소를 갖고 있는 파일
    - 사용할 파일이 이미 작성되어 있음
        - 레포지토리 복제해 Vagrant 파일 액세스 권한을 가지고 이를 이용해 VM 스핀업 
        - `git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git`

- clone 받은 파일로 이동
    - `cd certified-kubernetes-administrator-course/`

- `ls` 명령어 수행 시 Vargrantfile 존재해야 함
- `vi Vagrantfile`
    - Vagrant의 장점은 명령 하나만 실행하면 정확히 동일한 구성으로 VM 전체를 불러올 수 있다는 점
- `vagrant status` 
    - VM의 상태 확인 명령어
    - 현재 kubemaster, kubenode01, kubenode02 VM 존재
    - 하나는 master node, 두개는 worker node
    - 아직 VM을 안 불러왔으므로 "not created" 상태


- `vagrant up`
    - VM 3개 모두 Vagrant 파일의 정확한 명세로 프로비전
    - ubuntu/bionic64 이미지 import 후 다양한 VM 프로비전
    - 이 과정은 모든 VM을 켜고 실행하기 때문에 시간이 오래 소요

- `vagrant status`
    - 3개의 VM 모두 "running" 상태

- `vagrant ssh [연결할 node]`
    - 생성된 node와 연결
    - Ex. `vagrant ssh kubemaster`

- (kubemaster에 접속한 상태) `ls -la`
    - 실제로 kubemaster에 존재함을 알 수 있음

- (node에서 나가고 싶은 경우) `logout`
    - 로컬 컴퓨터로 돌아감


## Demo - Deployment with Kubeadm

| VM 생성 후 Kubernetes Cluster 생성

#### Kubeadm으로 Kubernetes Cluster 생성

| `kubernetes.io`에서 "Installing kubeadm" 문서 내용

- 이전에 VM 생성했으므로 "Installing a container runtime" 참고

1. Installing a container runtime

- **모든 Node에 Container Runtime 설치**

    | container runtime 하이퍼링크 클릭해 다른 페이지로 이동
    
    1. Forwarding IPv4 and letting iptables see bridged traffic

    - 모든 node에 복사 붙여넣기

    2. 모든게 실행 중인지 확인

    - 모든 node에 복사 붙여넣기
    - `lsmod`로 시작하는 명령어 2개

    3. 1로 설정했는지 확인

    - 모든 node에 복사 붙여넣기

- 우리는 containerd 사용
    | containerd 하이퍼링크 클릭해 다른 페이지로 이동

    1. getting started with containerd 하이퍼링크로 github 이동

    - Option2. From apt-get or dnf 사용
        - Ubuntu 선택
        - docker 페이지로 이동

    2. Set up the repository - 1.Update the apt package and install packages to allow apt to use a repository over HTTPs

    - 3개의 node에 복사 붙여넣기

    ```
    sudo apt-get update
    sudo apt-get install \\
        ca-certificates\
        curl\
        gnupg\
        lsb-release
    ```

    3. Set up the repository - 2. Add Docker's official GPG key

    - docker 공식 gpg 키 추가
    - 3개의 node에 복사 붙여넣기

    ```
    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```

    4. Set up the repository - 3. Use the following command to set up the repository

    - 저장소 준비
    - 3개의 node에 복사 붙여넣기

    ```
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

    5. apt update

    - 3개의 node에 복사 붙여넣기

    ```
    sudo apt update
    ```

    6. 최신 버전의 containerd 설치

    - docs에는 `sudo apt-get install docker-ce docker-ce-cli containerd.io ...` 와 같이 모든 docker compose 존재하지만, runtime만 있으면 되므로 containerd만 설치

    - 3개의 node에 복사 붙여넣기

    ```
    sudo apt install containerd.io
    ```

    7. 설치 확인


    ```
    systemctl status containerd
    ```

    8. cgroup drivers 

    - 중요 !
    - Linux 컴퓨터에 control group 존재
    - control group은 machine에서 실행되는 다른 프로세스에 할당된 자원 제한하는 데 사용
    - kubelet과 container runtime 둘 다 이 control group가 인터페이스해야 함
        - pod에 대한 다양한 리소스 관리를 시행하기 위해서
        - CPU, memory 제한 설정 등
    - control group과 인터페이스하려면 kubelet과 container runtime은 cgroup drive 사용해야 함
    - cgroup drivers는 `cgroupfs`와 `systemd` 존재
        - default는 cgroupfs
        - systemd를 사용할 때에는 systemd를 사용해야 하므로 변경 필요
        - kubelet이나 container runtime이 어떤 드라이버를 쓰든 상관 없지만 동일한 드라이버 사용해야 함
        - 우리는 둘 다 systemd 사용할 것
        - 


2. 


## Practice Test - Deploy a Kubernetes Cluster using Kubeadm

