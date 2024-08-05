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

3. 모든 node에 Container Runtime 설치

- containerd로 결정
- 해당 container runtime에 맞는 cgroup driver 설치

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

#### Kubeadm으로 Kubernetes Cluster 구성1 - Container Runtime 설정

| `kubernetes.io`에서 "Installing kubeadm" 문서 내용

- 이전에 VM 생성했으므로 "Installing a container runtime" 참고
- **Container Runtime 설정**

1. IPv4와 iptable로 트래픽 이동 확인

| Container Runtime 하이퍼링크 클릭해 다른 페이지로 이동
    
- Forwarding IPv4 and letting iptables see bridged traffic

    - 모든 node에 복사 붙여넣기

    ```
    # sysctl params required by setup, params persist across reboots
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.ipv4.ip_forward = 1
    EOF

    # Apply sysctl params without reboot
    sudo sysctl --system
    ```

- 1로 설정했는지 확인

    - `sysctl net.ipv4.ip_forward`
    - 모든 node에 복사 붙여넣기

2. 원하는 container runtime 설치 

| 원하는 Container Runtime 하이퍼링크 클릭해 이동

- **모든 Node에 Container Runtime 설치**

- 우리는 containerd 사용

    | "containerd" 하이퍼링크 클릭해 다른 페이지로 이동 > "getting started with containerd" 하이퍼링크로 github 이동
    
    - Option2. From apt-get or dnf 사용
        - "Ubuntu" 하이퍼링크 클릭해 docker 페이지로 이동
        - containerd를 설치하는 과정은 docker engine을 설치하는 것과 비슷

    1. Set up Docker's apt repository
    - docker 공식 gpg 키 추가
    - 3개의 node에 복사 붙여넣기

    ```
    # Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    # Add the repository to Apt sources:
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```

    2. Docker package 설치
    - containerd만 설치해도 됨
    - 3개의 node에 복사 붙여넣기

    ```
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

    3. 설치 확인

    ```
    systemctl status containerd
    ```

    4. `cgroup drivers` - 중요
    
    | Container Runtimes 페이지 

    - Linux 컴퓨터에 control group 존재
    - control group은 machine에서 실행되는 다른 프로세스에 할당된 자원 제한하는 데 사용
    - kubelet과 container runtime 둘 다 이 control group가 인터페이스해야 함
        - pod에 대한 다양한 리소스 관리를 시행하기 위해서
        - CPU, memory 제한 설정 등
    - control group과 인터페이스하려면 kubelet과 container runtime은 cgroup drive 사용해야 함
    - cgroup drivers는 `cgroupfs`와 `systemd` 존재
        - **default는 cgroupfs**
        - systemd를 사용할 때에는 systemd를 사용해야 하므로 변경 필요
        - kubelet이나 container runtime이 어떤 드라이버를 쓰든 상관 없지만 동일한 드라이버 사용해야 함
        - 우리는 둘 다 systemd 사용할 것
    - init program(=cgroup driver) 보려면 `ps -p 1`
        - process id가 1인 프로세스 조회
    - **systemd 설정 방법**
        
        | Container Runtimes 페이지에서 containerd 아래의 "Configuring the systemd cgroup driver" 부분

        1. Configuring the systemd cgroup driver
        - `sudo vi /etc/containerd/config.toml`
            - 기본 설정 모두 삭제 후 아래 명령어 붙여넣기
        ```
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
            [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
                SystemdCgroup = true
        ```

        2. 재시작
        - `sudo systemctl restart containerd`
        - 위 방식을 모든 node에서 진행
        - 모두 완료하면 Runtime 준비 완료


3. kubeadm, kubelet and kubectl 설치

| Installing kubeadm 페이지로 돌아오기

- `kubeadm`: cluster 부트스트랩을 책임지는 도구
- `kubelet`: 모든 cluster에서 작동하는 구성 요소. Pod와 Container 시작하는 일
- `kubectl`: command line utility로, cluster와 통신할 때 사용

- 구성

    | 모든 node에서 실행

    1. apt 패키지 업데이트
    
    ```
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg
    ```

    2. public sign key 다운로드
    ```
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    ```

    3. Kubernetes apt repository 추가

    ```
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

    4. 버전 업데이트
    ```
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    ```

=> 이제 Kubernetes cluster 구성 및 프로비저닝 시작 가능

#### Kubeadm으로 Kubernetes Cluster 구성2

| `kubernetes.io`에서 Creating a cluster with kubeadm" 문서 참고 (Installing kubeadm 다음 단계 페이지)

1. Initializing your control-plane node 

| Master node에서만 진행 !

- 동작
    1. (선택) control plane endpoint가 다수의 master node인 경우에만 실행
    - 현재는 건너뜀
    2. Pod Networking 기능
    - 사용할 pod networking 제공
    - `--pod-network-cidr` flag 전달
    3. (선택) kubeadm이 자동으로 사용하는 container runtime을 감지
    - 감지할 수 없는 경우 `--cri-socket` flag 전달
    4. (선택) 기본적으로 apiserver가 듣는 주소가 무엇인지 지정
    - master node에 설정한 올바른 고정 IP로 설정할 것
    - 궁극적으로 해당 IP 주소에 Worker node 모두가 apiserver로 액세스 가능
    - `kubeadm init` 명령어를 실행할 때 `--apiserver-advertise-address` flag 전달
        - 이때 우리가 설정한 특정 master node IP 주소 전달
    5. 이 모든 flag(특히 pod-network-cidr과 apiserver-advertise-address)를 가진 채 `kubeadm init` 실행

    - 따라서 `sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.56.2`
        - `ip addr`로 apiserver-advertise-address에 넣을 apiserver IP 주소 찾기

    6. 위에서 `sudo kubeadm init` 명령 이후 나오는 명령어 복사 붙여넣기
        - 홈 디렉토리에 .kube 폴더 생성 후 해당 폴더에 복사
        - Kubernetes cluster에 연결할 수 있도록 함
        - `kubectl get pods`로 연결 확인

    7. Pod-network 설정
        - `sudo kubeadm init` 명령 이후의 addons 링크 들어가기 
        - Weave net 사용
        - Weave net 페이지의 Installation 명령어 실행
            - `kubectl get pods -A` 시 배포되어있는 상태여야 함
         
        - 위에서 cluster cidr을 설정했으면 weanet의 IPALLOC_RANGE와 일치시켜야함 
            - `kubectl edit daemonset weave-net -n kube-system`
            - container 의 env에 추가
            ```
            - name: IPALLOC_RANGE
              value: 10.244.0.0/16  #pod network cidr로 넘겨준 IP 주소
            ```

2. Worker node 조인

    1. Worker node에 가서 `sudo kubeadm init` 명령 이후의 `kubeadm join` 명령어 복사 붙여넣기
        - Worker node를 cluster에 조인
        - 위 명령어가 실행되지 않으면 앞에 `sudo` 붙이기

    2. 확인
    - master node에서 `kubectl get nodes`

- (확인할 것!) 버전에 따른 kubelet cgroup driver
    | Installing kubeadm 페이지 > Container Runtimes > configuring a cgroup driver

    - cgroup driver 버전이 1.22 이하이면 kubelet cgroup driver 기본이 cgroupfs, 이상이면 systemd
    - 이전 버전이라 cgroup driver가 cgroupfs로 설정되어 있으면 아래 yaml 파일 구성 
    `kubeadm-config.yaml`
    ```
    kind: ClusterConfiguration
    apiVersion: kubeadm.k8s.io/v1beta3
    kubernetesVersion: v1.21.0
    ---
    kind: KubeletConfiguration
    apiVersion: kubelet.config.k8s.io/v1beta1
    cgroupDriver: systemd
    ```

    - `kubeadm init --config kubeadm-config.yaml`

## Practice Test - Deploy a Kubernetes Cluster using Kubeadm


Q1. kubeadm과 kubelet을 1.30.0-1.1 버전으로 설치

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- 4단계의 `sudo apt-get install -y kubelet kubeadm kubectl`에서 버전 지정
    - `sudo apt-get install -y kubelet=1.30.0-1.1 kubeadm=1.30.0-1.1 kubectl=1.30.0-1.1`


Q5

`ifconfig`

- 네트워크 인터페이스 설정하거나 IP 주소, 서브넷마스크, MAC 주소, 네트워크 상태 확인 가능
- 암기

`kubeadm init --apiserver-advertise-address=192.15.114.12 --pod-network-cidr=10.244.0.0/16 --apiserver-cert-extra-sans=controlplane`

- apiserver-advertise-address는 `ipconfig`에서 문제에 주어진 interface의 IP 주소 

- 아래 명령어 실행해 kubeconfig 파일 설정 완료할 것

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Q7

- Master node에서 `kubeadm init` 명령어 완료 시 나타난 메세지 중 join 복사 후 Worker node에 붙여넣기

Q8

- `kubeadm init` 시 addon 링크에서 flannel 찾아 배포
    - `kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml`

- flannel이 eth0 인터페이스와 상호작용하도록 수정
    - `k edit daemonset [DAEMONSET NAME]`
    - args에 `- --iface=eth0` 추가