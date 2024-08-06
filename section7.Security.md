## Kubernetes Security Primitives

- Kubernetes는 production application을 호스팅하는 플랫폼으로, 보안이 중요

#### Secure Hosts

| Cluster를 형성하는 Host부터 시작

- Host에 대한 모든 액세스는 보안되어야 함
  - Password based authentication은 사용할 수 없고, SSH Key based authentication만 사용 가능
- Kubernetes host는 물리적 또는 가상 인프라를 보호하기 위해 필요한 다른 수단이 있음

#### Secure Kubernetes

| Cluster 보안을 위해 어떤 위험과 조치를 취해야 하는가?

- Kubernetes 내의 모든 동작의 중심은 `kube-apiserver`
  - kube control utility나 API에 직접 액세스함으로써 cluster에서 거의 모든 작업 수행 가능 => 이것이 1차 방어선
  - **API Server 자체에 대한 액세스 제어**

1. Who can access?

- 누가 API Server에 액세스할 수 있는가 => Authentication

2. What can they do?

- 그들이 무엇을 할 수 있는가 => Authorization

#### Authentication

| Who can access?

- API Server에 인증할 수 있는 여러 방식 존재

- Files - Username and Passwords
- Files - Username and Tokens
- Certificates
- External Authentication Providers - Ex.LDAP
  - 외부 인증 공급자와도 통합 가능
- Service Accounts
  - 컴퓨터는 Service Account 생성

#### Authentication

| Cluster에 액세스 권한을 얻은 후(Authentication 획득) What can they do?

- RBAC Authorization
  - 사용자가 특정 권한으로 그룹과 연결
- ABAC Authorization
  - 특성 기반 액세스 제어
- Node Authorization
- Webhook Mode

#### TLS Certificates

- Cluster 내의 모든 통신 구성 요소 간에는 ETCD Cluster, Kube Controller Manager, Kube Scheduler, Kube API Server 존재
- Worker node에서 실행되는 Kubelet과 kube-proxy는 TLS 암호화로 보안

#### Network Policy

- Cluster 내의 Application 간의 통신은 기본 설정 상 모든 pod는 무리 내의 다른 pod에 접근 가능 => Network Policy 사용해 액세스 권한 제어 가능

## Authentication

- Kubernetes Cluster는 다중 Node와 물리적, 가상적, 그리고 다양한 구성 요소로 함께 작동

  - Admin(관리자): User는 관리 업무를 수행하기 위해 Cluster에 액세스
  - Developer(개발자): 앱을 테스트하거나 배포하기 위해 cluster에 액세스
  - End User: cluster에 배포된 application에 액세스
  - Bots: 통합 용도로 cluster에 액세스하는 타사 애플리케이션

- 인증과 승인 매커니즘을 통해 cluster에 대한 관리 액세스를 보안함으로써 내부 구성 요소 간의 통신을 보안

| 이번 섹션에서는 Authentication 매커니즘을 통해 Kubernetes cluster에 액세스 권한을 확보하는 데 초점

#### Accounts

- Cluster에 액세스할 수 있는 다양한 사용자 => Admins, Developers, Application End Users, Bots

  - 단, Application End User는 Application 자체에 의해 내부적으로 보안을 다루어야 하기에 제외
  - 관리 목적으로 사용자가 Kubernetes cluster에 액세스하는 것에 초점을 둠

- 유형별로 User(Admins, Developers)와 Service Account(Bots)로 나뉨

  - `User`

    - Kubernetes는 사용자 계정을 직접 관리하지 않고, 외부 외부 소스에 의존
    - 사용자 세부 정보, 인증서가 있는 파일, LDAP 같은 타사 ID 서비스가 사용자 계정 관리
    - Kubernetes cluster에서 user를 생성하거나 user list를 볼 수 없음

  - `Service Account`
    - Kubernetes가 관리할 수 있음
    - Kubernetes API를 사용해 Service Account 생성하고 관리할 수 있음

#### Accounts (=User)

- 모든 User access는 `kube-apiserver`에 의해 관리

  - kubectl이나 API를 통해 직접 cluster에 액세스하든 아니든
    - `kubectl`
    - `api`: curl https://kube-server-ip:6443/

- kube-apiserver
  1. kube-apiserver는 요청을 처리하기 전에 Authenticate User
  2. Proces Request

#### Auth Mechanisms

| Kube-apiserver가 인증하는 방법

- 구성될 수 있는 다양한 Authentication 매커니즘 존재

1. Static Password File

- Username과 Password

2. Static Token File

- Username과 Token

3. Certificates

4. Identity Services

- LDAP나 Kerberos와 같은 타사 Authentication protocol

#### Auth Mechanisms - Basic (Static Password File, Static Token File)

1. Static Password File

- CSV 파일에 User list와 Password 저장하고, 이를 사용자 정보 소스로 이용

  - 파일은 3개의 열과 password, username, user id로 구성 - CSV 파일에서 선택적으로 4번째 열에 group을 지정해 할당할 수 있음
    `user-details.csv`

  ```
  password123,user1,u0001
  password123,user2,u0002
  password123,user3,u0003
  #선택적으로
  password123,user4,u0004,group1
  ```

- 파일 이름을 kube-apiserver 옵션으로 넘김

  - 방법1. kubeadm으로 cluster 구성하지 않은 경우, `kube-apiserver.service` 파일에 `--basic-auth-file=user-details.csv` 추가
    - kube-apiserver 재시작
  - 방법2. kubeadm으로 cluster 구성한 경우, kube-apiserver의 Pod YAML 파일(/etc/kubernetes/manifests/kube-apiserver.yaml) 수정
    - `--basic-auth-file=user-details.csv`
    - YAML 파일 업데이트하면 자동으로 kube-apiserver 재시작

- Basic Auth Mechanism을 이용한 경우, `-u "[USERNAME]:[PASSWORD]"`를 통해 apiserver에 액세스 가능
  - `curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password"`

2. Static Token File

- CSV 파일
  `user-token-details.csv`

  ```
  fkHIkjkfdlsaihgrj49vsk,user10,u0010,group1
  kl8ew0jnreKBvdsfqergjl,user11,u0011,group2
  ```

- kube-apiserver 옵션으로 넘김

  - 위의 방식과 동일
  - 옵션은 `--token-auth-file=user-token-details.csv`

- `--header "Authorization": Bearer [TOKEN]` apiserver에서 액세스
  - `curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer fkHIkjkfdlsaihgrj49vsk"`

#### NOTE

- 위 방식은 Username, Password, Token을 명확한 텍스트에 저장하는 방식으로, authentication mechanism으로 권장하지 않음
- kubeadm 설정에서 auth file을 제공할 때에는 volume mount를 고려해야 함

## TLS Basics

| 인증서는 거래 도중 상호 신뢰를 보장하기 위해 사용

#### Demo-WebServer

**Scenario1**

- 사용자가 웹 서버에 액세스하려하는 경우, TLS 인증서는 사용자와 서버 사이의 통신이 암호화되도록 함
- 안전한 연결성이 없으면 사용자가 온라인 뱅킹 앱에 접속할 경우 입력한 자격 증명(Credential)이 일반 텍스트 형식으로 전송
- 해커가 네트워크 트래픽을 탐지하면 Credential을 쉽게 추출해 사용자의 은행 계좌 해킹 가능
- 따라서 전송되는 데이터 암호화

- Key로 데이터를 암호화했는데 key는 무작위의 숫자와 알파벳 조합
- 데이터에 임의의 숫자를 추가하고 인식할 수 없는 포맷으로 암호화하여 서버로 전송
- 서버는 key 없이는 데이터를 해독할 수 없으니 key의 복사본도 서버로 전송해 서버가 메세지를 해독하고 읽을 수 있게 함
  - key도 같은 네트워크로 전송되니까 해커도 데이터를 해독할 수도 있음 => `Symmetric Encryption`
    - 안전한 암호화 방식
    - 데이터를 암호화하고 해독하는 데 같은 key를 사용하고, 그 key는 수신기와 교환해야 하기 때문에 해커가 접근해 데이터를 해독할 위험 존재 => `Asymmetric Encryption` 등장
- 즉 문제는 데이터를 암호화하는 데 사용된 key가 암호화된 데이터와 함께 네트워크를 통해 서버로 전송된다는 점 => 키를 안전하게 빼낼 수 있으면 Symmetric Encryption으로 안전하게 통신 가능 => 이때 Asymmetric Encryption 사용하면 가능

**해결과정**

1. Public key와 Private key 생성

- openssl 명령어 사용해 **Asymmetric Encryption**을 위한 key 생성
- `openssl genrsa -out my-banck.key 1024`로 private key, `openssl rsa -in my-bank.key -pubout > mybank.pem`로 public key

2. 사용자가 HTTPS를 이용해 웹 서버에 처음 액세스하면 서버에서 Public key 받음

- 이때 해커도 Public key의 복사본 받았다고 가정

3. 사용자의 브라우저가 서버가 제공한 Public key(Asymmetric Public Lock)를 이용해 Symmetric key 암호화

- Symmetric key는 안전 => Asymmetric Private Key로만 해독할 수 있음

4. 사용자는 암호화된 Public key와 암호화된 Symmetric key를 서버로 전송

- 해커도 복사본 받음

5. 서버는 Private key(Asymmetric Private Key)로 메세지를 해독하고 Symmetric key 회수해 완벽히 해독

- 해커는 암호화된 메세지와 어떤 데이터도 해독할 수 없는 Public Key(=Public Lock)만 남음
- 해커는 암호를 풀 Private key가 없고, 메세지에서 Symmetric key를 해독할 key도 없음
- 해커가 가진 Public key만 메세지를 잠그거나 암호화할 수 있고 해독할 수는 없음
- Symmetric key는 사용자와 서버만 안전하게 사용할 수 있음

=> 이 과정을 통해 데이터를 안전하게 암호화해 전송할 수 있음

| 해커는 계정을 해킹할 새로운 방법 찾음

**Scenario2**

- Credential을 얻으려면 서류에 타자로 쳐야 함을 깨닫고 은행 웹사이트와 똑같은 웹사이트를 만듬
  - 디자인, 그래픽, 웹사이트도 실제 웹사이트의 복제품
- 자기 서버로 웹사이트 운영하고 Public key와 Private key 쌍을 직접 만들어 웹 서버에 구성
- 사용자의 환경이나 네트워크를 조정해 요청을 자기 서버로 보내는 데 성공
- 사용자가 웹사이트 주소를 입력하면 은행 로그인 페이지가 나타나 이름과 암호 입력
- 위에서 진행한 방식으로 안전하게 전송했지만, 해커의 서버로 전송 => 문제 발생

**해결과정**

| 사용자가 서버에서 받은 key를 보고 진짜 은행 Server에서 나온 합법적인 key인지 확인한다면 문제 해결 가능

- 서버가 key를 보낼 때 key를 보내는 것이 아니라 key를 가진 인증서(certificate) 전송
  - 인증서를 자세히 보면 실제 인증서와 비슷한 디지털 포맷
  - 누구에게 발급되는지, 해당 서버의 퍼블릭 키, 해당 서버의 위치 등에 대한 정보 담음
- 웹 서버에 대한 것이라면 사용자가 브라우저의 URL에 입력한 것과 일치해야 함

  - 인증서는 누구나 만들 수 있지만, 해당 인증서가 진품인지 판별해야 함
  - 인증서 생성 후 직접 서명해야 하는데 이 Self Sign을 보고 진위여부 판별
  - 이 판별은 브라우저가 해줌
    - 모든 웹 브라우저는 인증서 유효성 검사 매커니즘이 기본으로 제공되고, 브라우저 텍스트에서는 서버에서 받은 인증서가 합법적인지 유효성 확인
    - fixed 인증서임을 식별하면 실제로 경고를 줌
  - 웹 브라우저가신뢰할 합법적인 웹 서버 인증서 만드는 방법

    - 권한이 있는 사람(CA.Certificate Authority)이 인증서에 Sign
      - CA 예시: Symantec, GlobalSign, digicert
    - 1. 인증서 생성 => `Certificate Signing Request(CSR)` 전송
      - 생성한 key와 웹사이트의 domain 이름 이용
      - 또는 `openssl req -new -key my-bank.key -out my-bank.csr -subj "/C=US/ST=CA/O=MyOrg,Inc./CN=my-bank.com"`
        - my-bank.csr 생성
        - CSR 파일은 인증서 서명 요청으로 CA에 보내져 서명 받아야 함
    - 2. `Validate Information`
      - CA 관계자가 상세 사항을 확인
      - 해커도 같은 방법으로 인증서에 서명을 받으려하면, 인증서 유효성 검사 단계에서 실패해 인증서 거부
      - CA는 도메인의 실제 소유주를 확인하기 위해 특별한 기술 사용
    - 3. `Sign and Send Certificate`

      - 확인되면 인증서에 서명해서 다시 보내줌

    - 가짜 CA 서명을 브라우저가 아는 방법
      - 각 CA는 Public key와 Private key 쌍 존재
      - 브라우저에 모든 CA의 Public key가 기본으로 제공되고, 인증서에 서명할 때 Private key 사용
      - 브라우저는 CA의 Public key를 사용해 CA가 직접 서명한 인증서임을 확인

- 방문하는 공공 웹사이트가 은행, 이메일 등 합법적인지 확인하는 공용 CA로, 개별적으로 호스팅된 사이트의 유효성을 검사하는 것을 돕지는 않음

**동작과정**

1. 서버
   - 관리자는 한 쌍의 key로 서버 SSH 연결 보안
   - 서버는 한 쌍의 key로 HTTPS 트래픽 확보
   - 서버는 CA에 인증서 서명 요청(CSR) 전송
2. CA
   - CA는 private key로 CSR에 서명
     - 모든 사용자는 CA Public key의 복사본을 가짐
3. 서버
   - 서명된 인증서는 서버로 다시 보내짐
   - 서버는 서명된 인증서로 웹 응용 프로그램 구성
4. 서버 <-> 사용자(브라우저)
   - 사용자가 웹 응용 프로그램에 액세스할 때마다 서버는 먼저 Public key로 인증서 보냄
   - 브라우저가 인증서를 읽고 서버의 Public key를 유효성 검사와 검색하기 위해 CA Public key 사용
   - 대칭 키를 생성해 모든 통신에 사용
5. 사용자
   - 대칭 키는 서버의 Public key로 암호화되어 서버로 전송
6. 서버
   - 서버는 개인 키로 메세지 해독하고 대칭 키 회수
     - 대칭 키는 앞으로의 통신을 위한 것
   - 관리자가 SSH 보안 키 페어 생성
   - 웹 서버는 HTTPS로 웹사이트를 보호할 키 페어 생성
   - 인증서 기관은 자체 관리자를 생성해 인증서에 서명
7. 사용자
   - 최종 사용자는 단일 대칭키만 생성
   - 웹사이트에서 신뢰를 설정하면 아이디와 암호로 웹 서버에 인증
   - 서버 키퍼인 클라이언트는 서버의 유효성 확인
     - 이때 서버는 클라이언트인지 해커인지 알 수 없음 => 클라이언트가 맞는지 서버측에서 확인하는 방법 => 트러스트 구축 연습의 일부로 **서버가 클라이언트에게 인증서 요청**
     - 클라이언트는 한 쌍의 키와 서명된 인증서를 유효한 CA로부터 생성
     - 클라이언트는 인증서를 서버로 전송하고 클라이언트가 자신이 말하는 사람이 맞는지 확인

=> 웹 서버에서는 TLS 클라이언트 인증서가 구현되지 않고, 호스트 아래에 구현되어 있음 - 일반 사용자는 인증서를 수동으로 생성하고 관리할 필요 없음

- 전체 인프라는 Public Key Infrastructure(PKI)로 알려져 있음

#### Naming Convention

- Public key가 있는 인증서 => `*.crt`, `*.pem`

  - Ex. server.crt, server.pem, client.crt, client.pem

- Private key가 있는 인증서 => `*.key`, `-key.pem`
  - Ex. server.key, server-key.pem, client.key, client-key.pem

#### Symmetric Encryption vs Asymmetric Encryption

1. Symmetric Encryption

- 안전한 암호화 방식
- 데이터를 암호화하고 해독하는 데 같은 key를 사용하고, 그 key는 수신기와 교환해야 하기 때문에 해커가 접근해 데이터를 해독할 위험 존재 => `Asymmetric Encryption` 등장

2. Asymmetric Encryption

| 사용자가 공개키(public key)로 데이터를 암호화하고, 서버가 개인키(private key)로 데이터를 복호화 함

- 데이터를 암호화하고 해독하는 데 단일 key를 쓰는 Symmetric Encryption과 달리, 한 쌍의 private key와 public key 사용 => Private Key & Public Lock
- Key는 자신만 가지고 있는 Private Key이고, Lock은 누구나 접근할 수 있는 Public Lock
  - Key는 다른 사람과 공유하면 안 됨
  - Lock은 공개되어 있고 다른 사람과 공유할 수도 있지만, 해당 Key가 없는 경우 해결할 수 없음
- Public Lock으로 데이터를 암호화하여 잠그는 경우, 관련 Key로만 열 수 있음

- Key pair를 이용해 서버에 대한 SSH 액세스를 보안하는 사례

  - 접근이 필요한 서버 존재하고 key 쌍 사용
  - 1. `ssh-keygen` 명령어로 Public key와 Private key 쌍 생성
    - `id_rsa`와 `id_rsa.pub` 생성
    - id_rsa는 private key, id_rsa.pub은 public key(=Public Lock)
  - 2. 서버 접근 권한을 모두 잠가 서버를 안전하게 함
    - Public Lock 문만 존재
    - `cat ~/.ssh/authorized_keys`로 public key로 entry 추가
  - 3. 개인 노트북에 private key의 위치 지정
    - `ssh -i id_rsa_user1@serve`
  - 동일한 환경에 다른 서버가 존재하는 경우, 여러 서버로부터 key pair 보호하는 방법
    - Public Lock 복사본을 만들어 원하는 만큼 서버에 둘 수 있음
    - SSH에 대한 동일한 Private key를 모든 서버에 안전하게 사용할 수 있음
  - 다른 사용자가 해당 서버에 접속해야 하는 경우

    - 다른 사용자도 Private key와 Public lock을 생성
    - 서버에 다른 사용자를 위한 추가 문을 만들어 공용 잠금장치로 사용 가능
    - 모든 서버에 복사하면 다른 사용자가 private key를 이용해 서버에 접속 가능

    ![alt text](image-64.png)

- Public key만 암호화하고 Private key만 해독하는 것이 아니라, 한 쌍으로 동작해 한 쪽이 암호화하면 한 쪽이 해독

  - BUT Private key로 암호화하면 Public key를 이용해 누구나 암호를 풀고 메세지를 읽을 수 있음

- Asymmetric encryption 통신 과정

![alt text](image-142.png)

![alt text](image-143.png)

## TLS in Kubernetes

#### 이전 섹션 정리

- Server (Certificates)

  - Public Key와 Private key

- CA (Certificate Authority)

  - Server Certificates에 서명할 때 Root Certificates(루트 인증서) 필요

- Client (Certificates)

  - Server가 Client에게 Client Certificates를 이용해 자신을 검증해달라고 요청

- 따라서 Server Certificates(Server), Root Certificates(CA), Client Certificates(Client) 총 3개의 인증서 구성

- Naming Convention

  - Public key가 있는 인증서 => `*.crt`, `*.pem`

    - Ex. server.crt, server.pem, client.crt, client.pem

  - Private key가 있는 인증서 => `*.key`, `-key.pem`
    - Ex. server.key, server-key.pem, client.key, client-key.pem
    - 보통 private key는 이름에 "key"라는 단어 존재

#### TLS in Kubernetes

- Kubernetes Cluster에는 Master node와 Worker node 존재
- Node 사이의 통신은 보안이 필요하고 암호화되어야 함
- 모든 Service와 고객간의 상호작용은 안전해야 함
- Kubernetes cluster 내 모든 구성 요소 간의 통신도 보안 필요
- 2가지 주요 요구 사항
  - Server Certificates for Servers
  - Client Certificates for Clients

#### Server Certificates for Servers

1. kube-apiserver Server

- cluster 관리 목적으로 HTTPS 서비스 제공
  - 다른 구성 요소와 외부 사용자와 함께
- kube-apiserver는 Server이고, Client와의 모든 통신을 보호하기 위해 Certificates 필요 => `apiserver.crt`, `apiserver.key`
  - Naming convention에 따라 전자는 public key, 후자는 private key
  - 이 Certificates 이름은 kubernetes 설정에 따라 다를 수 있음

2. ETCD Server

- cluster에 관한 모든 정보 저장하므로 key 쌍 필요 => `etcdserver.crt`, `etcdserver.key`

3. Kubelet Server

- Worker node의 kubelet도 key 쌍 필요 => `kubelet.crt`, `kubelet.key`
- kube-apiserver가 worker node와 상호 작용하기 위해 통신하는 HTTPS API endpoint

#### Client Certificates for Clients

| 각 Server에 액세스하는 Client 관련

1. kube-apiserver를 Server로 가지는 Client

- 1. Admin
  - kube-apiserver와 kubectl REST API 통신
  - `admin.crt`, `admin.key`
- 2. Kube-scheduler

  - kube-apiserver에 보고해 스케줄이 필요한 pod 탐색 -> kube-apiserver가 올바른 worker node의 pod 찾음
  - `scheduler.crt`, `scheduler.key`

- 3. Kube-controller Manager

  - `controller-manager.crt`, `controller-manager.key`

- 4. Kube-proxy
  - `kube-proxy`, `kube-proxy.key`

2. ETCD Server를 Server로 가지는 Client

- Kube-apiserver
  - kube-apiserver는 etcd server와 통신하는 유일한 server
  - 방법1. kube-apiserver는 고유의 API service를 처리하기 위해 사용한 key와 동일한 key를 사용할 수 있음
  - 방법2. kube-apiserver는 ETCD Server와 통신하기 위해 새로운 인증서 한 쌍을 생성해 ETCD Server에 인증할 수 있음 => `apiserver-etcd-client.crt`, `apiserver-etcd-client.key`

3. Kubelet Server를 Server로 가지는 Client

- 1. Kube-apiserver

  - 개별 node에 있는 kubelet server와 통신해 Worker node 모니터링
  - ETCD Server와 동일하게 기존의 key를 사용해도 되고 새로운 key 쌍을 만들 수도 있음 => `apiserver-kubelet-client.crt`, `apiserver-kubelet-client.key`

- 2. kubelet
  - kubelet과 서로 통신 => `kubelet-client.crt`, `kubelet-client.key`

#### Certificate Authority (CA)

- Client Certificates와 Server Certificates에 서명하려면 CA(Certificate Authority) 필요

- Kubernetes는 cluster에 적어도 하나의 인증서 인증 요구
- 현재는 Cluster를 위해 CA 하나만 사용
- CA는 자체 Certificates와 key 존재 => `ca.crt`, `ca.key`

## TLS in Kubernetes - Certificate Creation

- Certificate를 생성하기 위해 다양한 도구 사용 가능
  - `EASYRSA`, `OPENSSL`, `CFSSL`
  - OpenSSL 사용해 생성할 것

#### CA Certificate

- `ca.crt`와 `ca.key` 생성

1. Generate keys
- OpenSSL 명령어를 통해 private key 생성
- `openssl genrsa -out ca.key 2048` => ca.key

2. Certificate Signing Request

| CA로부터 공식적인 인증서를 받기 위한 신청서 생성 단계로, 신청서 양식이 CSR

- 방금 만든 key로 인증서 서명 요청 생성
  - 인증서 서명 요청은 모든 세부 사항 인증서 같은 것. BUT 서명은 없음
  - 인증서 서명 요청에서 구성 요소의 이름 지정
  - CN(Common Name) 필드의 Kubernetes CA 인증서임을 나타냄
- `openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr` => ca.csr

3. Sign Certificates

- 인증서에 서명
  - 이전에 생성한 인증서 서명 요청을 명시화
  - 첫 단계에서 생성된 private key를 이용해 CA가 자체 서명
- `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt` => ca.crt
- 다른 모든 인증서에는 현재 생성한 CA key pair를 사용해 서명
  - CA는 private key와 root certificate file 존재

#### Admin User - Client Certificate

- `admin.crt`와 `admin.key` 생성
- 이름: kube-admin

1. Generate keys
- private key 생성
- `openssl genrsa -out admin.key 2048` => admin.key

2. Certificate Signing Request
- csr 생성해 admin 사용자 이름 지정
  - 아래 명령어에서는 kube-admin
  - kubectl 명령 실행 시 kubectl client 인증을 위한 이름
  - `openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr` => admin.csr

3. Sign Certificates
- ca.crt와 ca.key, 즉 **CA key pair로 인증서에 서명**
- `openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt` => admin.crt
- 이후 cluster 내 유효한 인증서가 됨
  - 관리자가 kubernetes cluster에 인증할 때 사용하는 인증서
- key와 인증서를 생성하는 과정은 새 사용자에 대한 사용자 계정을 만드는 것과 유사
  - 인증서는 유효한 사용자 ID
  - key는 암호
- 이 사용자와 다른 사용자 구별하는 방법
  - 이 사용자는 기본 사용자가 아닌, Admin 사용자로 구별되어야 함
  - => 인증서에 사용자에 대한 Group 세부 정보 추가
    - 2번째 단계에 `/O=system:masters` 명령어 추가
    - Group: SYSTEM:MASTERS
      - Kubernetes에는 관리 권한을 가진 SYSTEM:MASTERS 그룹 존재
- Kube-apiserver에 접근하는 다른 구성 요소에 대한 Client Certificate를 생성하는 프로세스는 모두 동일

  1. Kube-scheduler
  - kube-scheduler는 system 구성 요소로 이름에 system을 붙여야함
  - 이름: system:kube-scheduler
  2. Kube-controller-manager
  - 이름: system:kube-controller-manager
  3. Kube-proxy
  - 이름: system:kube-proxy

#### Certificate 사용

- Cluster를 관리하기 위해 Admin Certificate 예시로 진행
- Kube-apiserver에 만드는 REST API 호출에서 사용자 이름과 암호를 대신해 인증서 사용 가능
  - option으로 key, cert, cacert 지정
  - `curl https://kube-apiserver:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt`
- 인증서 사용하지 않고는 모든 매개 변수를 kube-config.yaml 이름의 configuration file에 옮기는 것
  - 그 안에서 API server endpoint 세부 정보와 사용할 인증서 지정

  `kube-config.yaml`
  ```
  apiVersion: v1
  clusters:
  - cluster:
      certificate-authority: ca.crt
      server: https://kube-apiserver:6443
    name: kubernetes
  kind: Config
  users:
  - name: kubernetes-admin
    user:
      client-certificate: admin.crt
      client-key: admin.key
  ```

#### CA root Certificate

- Client가 Server에서 보낸 인증서 유효성 검사를 위해, 또는 그 반대의 경우 모두 CA와 공공 인증서의 사본 필요
  - Ex. Web Application의 경우 사용자 브라우저에 이미 설치되어 있음
- Kubernetes에서도 구성 요소끼리 서로를 확인하려면 CA의 root Certificate 복사본 필요
- 따라서 Server나 Client가 인증서를 갖고 구성할 때마다 CA root Certificate 지정해야 함

#### ETCD Server - Server Certificate

- ETCD 인증서를 생성하는 절차는 동일
- 이름: ETCD-Server, Key pair: etcdserver.key, etcdserver.crt
- ETCD Server는 고가용성 환경에서 다중 서버에 걸쳐 cluster로 배포될 수 있음
  - 그 경우 cluster 내 다른 멤버 간의 통신을 안전하게 하려면 `추가 peer Certificate를 생성`해야함
  - 이름: ETCD-peer, Key pair: etcdpeer1.key와 etcdpeer1.crt, etcdpeer2.key와 etcdpeer2.crt

- 인증서가 생성되면 ETCD Server를 시작하는 동안 지정
  - etcd yaml 파일에는 key-file, cert-file, peer-cert-file, peer-client-cert-auth, peer-key-file, peer-trusted-ca-file, trusted-ca-file 옵션 존재

#### Kube API Server - Server Certificate

- `apiserver.crt`와 `apiserver.key` 생성해야 함
- 이름: kube-api server
- API Server는 cluster 내 모든 구성 요소 중 가장 인기 있음
  - cluster 안에 많은 이름과 별칭 존재하며 모든 이름은 서버를 위해 생성된 인증서에 존재해야 함
    - 그래야 참조할 때 유효한 연결을 할 수 있음
  - kubernetes, kubernetes.default, kubernetes.default.svc, kubernetes.default.svc.cluster.local, 10.96.0.1(IP 주소), 172.17.0.87(서버를 실행하는 호스트 IP 주소)

1. Generate keys

- private key 생성
- `openssl genrsa -out apiserver.key 2048`

2. Certificate Signing Request

- kube-apiserver 이름 지정
- `openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr`

- 다른 이름 명시하는 방법 => `openssl 구성 파일 생성`
  1. `openssl.cnf`의 alt_names에 위에서 말한 이름과 별칭 모두 지정
  `openssl.cnf`
  ```
  ...
  [alt_names]
  DNS.1 = kubernetes
  DNS.2 = kubernetes.default
  DNS.3 = kubernetes.default.svc
  DNS.4 = kubernetes.default.svc.cluster.local
  IP.1 = 10.96.0.1
  IP.2 = 172.17.0.87
  ```

  2. 명령어에 -config 옵션 추가
  - `openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr -config openssl.cnf`

3. Sign Certificates

- `openssl x509 -req -in apiserver.csr CA ca.crt -CAkey ca.key -out apiserver.crt`

#### Kube-apiserver key 지정

- Kube-apiserver의 API Client Certificate 고려해야 함
  - etcd와 kubelet을 client로 통신하는 Certificate
  - `apiserver-kubelet-client.crt`와 `apiserver-kubelet-client.key`, `apiserver-etcd-client.crt`와 `apiserver-etcd-client.key`
- kubeapiserver executable or service configuration file에 지정되어야 함
  - 1. 모든 구성 요소가 client를 확인하기 위해 ca 인증서 필요
    - `--client-ca-file=/var/lib/kubernetes/ca.pem`

  - 2.API Server Certificate TLS 인증 옵션 아래에 지정
    ```
    --tls-cert-file=/var/lib/kubernetes/apiserver.crt
    --tls-private-key-file=/var/lib/kubernetes/apiserver.key
    ```
  
  - 3. kube-apiserver에서 사용되는 client certificate 지정
    - etcd server에 client로 연결
    ```
    --etcd-cafile=/var/lib/kubernetes/ca.pem
    --etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt
    --etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key
    ```
  - 4. kube-apiserver에서 사용되는 client certificate 지정
    - kubelet에 client로 연결
    ```
    --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem
    --kubelet-client-certificate=/var/lib/kubernetes/apiserver-kubelet-client.crt
    --kubelet-client-key=/var/lib/kubernetes/apiserver-kubelet-client.key
    ````

#### Kubelet Server - Server Certificate

- `kubelet.crt`와 `kubelet.key` 생성해야 함
- 이름: [NODE NAME]
  - Certificate 이름은 kubelet이 아닌 node name으로 지정
- kubelet server는 HTTPS API Server로 각 node에서 실행되며 node 관리에 책임 있음
- apiserver가 node를 모니터링하기 위해 통신하고 node에서 어떤 pod가 scheduling될지에 대한 정보 전송
- cluster의 각 node에 대해 key Certificate 쌍 필요

- 인증서 생성은 위와 동일
- 인증서 생성 후 kubelet-config.yaml 파일에 사용
  `kubelet-config.yaml` <- node01
  ```
  kind: KubeletConfiguration
  ...
  authentication:
    x509:
      clientCAFile: "/var/lib/kubernetes/ca.pem"
  ...
  tlsCertFile: "/var/lib/kubelet/kubelet-node01.crt"
  tlsPrivateKeyFile: "/var/lib/kubelet/kubelet-node01.key"
  ```

#### Kubelet Client Server - Client Certificate

- kubelet이 kube-apiserver와 통신할 때 사용하게 될 Certificate 존재 => `kubelet-client.crt`, `kubelet-key.key`
- Certificate 이름: system:node:[NODE NAME]
- kube-apiserver가 어떻게 올바른 사용 권한을 주는가?
  - (Admin에서 SYSTEM:MASTER Group이 추가된 것과 동일)
  - Certificate에 SYSTEM:NODES라는 Group 추가
- Certificate가 생성되면 kube-config 파일 지정됨

## View Certificate Details

- 기존 cluster에서 certificate를 보는 방법

- Cluster가 어떻게 설정되었는지 아는 것이 중요
  - 이에 따라 Certificate 생성과 관리도 다양
    1. Kubernetes cluster를 처음부터 스스로 배포
      - 스스로 인증서를 생성해야 함
      - 모든 구성 요소를 네이티브 서비스로 배포
      - `cat /etc/systemd/system/kube-apiserver.service`
    2. kubeadm 사용해 자동 프로비저닝
      - 모든 구성 요소를 pod로 배포
      - `cat /etc/kubernetes/manifests/kube-apiserver.yaml`

- kubeadm으로 가정하고 진행
- `cat /etc/kubernetes/manifests/kube-apiserver.yaml`
  1. 인증서 확인하고 메모 => ca 관련, etcd 관련, kubelet 관련, apiserver 관련 Certificate
  - client-ca-file, etcd-cafile, etcd-certfile, etcd-keyfile, kubelet-client-certificate, kubelet-client-key, tls-cert-file, tls-private-key-file
  2. 각각의 인증서 내부를 살펴 해당 인증서 detail 살펴보기
  - Ex. tls-cert-file 인증서 살펴볼것
  - `openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout`
    - 인증서 파일을 입력으로 제공해 인증서 디코딩하고 세부사항 조회
    - `Subject: CN=kube-apiserver` 확인
    - `X509v3 Subject Alternative Name`에 별명이 모두 존재하는지 확인
    - `Not After`로 만료기한 확인
    - `Issuer`로 발급자 확인. 증명서를 발급한 CA. kubeadm은 CA로 kubernetes 존재

#### Inspect Service Logs

1. 직접 cluster 구축한 경우

- `journalctl -u etcd.service -l`

2. kubeadm 사용해 구축한 경우

- `kubectl logs etcd-master`

- 가끔 kube-apiserver나 etcd server같은 핵심 구성 요소가 다운되면 kubectl 유틸리티 사용할 수 없음
  - docker 명령어 사용 
  - `docker ps -a`로 container ID 확인
  - `docker logs [CONTAINER ID]`

## Practice Test - View Certification Details

Q1. Identify the certificate file used for the kube-api server

- `k get pods -A`에서 pod로 배포되었으므로 kubeadm 사용한 것
- `k describe pod [CONTROLPLANE POD] -n kube-system`
  - `--tls-cert-file=/etc/kubernetes/pki/apiserver.crt` 옵션이 kube-api server certificate file

- 이때 key와 certificate 구분 !
  - key는 .key
  - certificate는 .crt

Q6. What is the Common Name (CN) configured on the Kube API Server Certificate?

`openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout`

- openssl에 -in 옵션 뒤에 certificate 경로 넣기

Q12

- kubectl이 동작하지 않는 경우 docker를 사용하는 경우는 `docker` command, cri-o를 사용하는 경우 `crictl` 명령어 사용
  - `docker ps -a` or `crictl ps -a`
  - 이때 방금 생성된 container, 즉 running 상태의 container log 확인 => "2379 port 번호로 etcd에 문제가 있음을 알아차림"
  - `docker logs [CONTAINER ID]` or `crictl logs [CONTAINER ID]`
- 오류 메세지 경로 찾기
  - `ls /etc/kubernetes/pki/etcd`에 server-certificate.crt 존재 X
- `/etc/kubernetes/manifests/etcd.yaml | grep server-certificate.crt`
  - server-certificate.crt가 존재하는 --cert-file을 올바르게 변경
- 시간 오래 소요

Q13

- `/etc/kubernetes/manifests/etcd.yaml`와 같이 /etc/kubernetes/manifests 아래에 구성 요소 yaml 파일 존재 

- kube-apiserver yaml 파일에는 ca가 2개 존재하는데 --client-ca-file은 kube-apiserver의 ca.crt로 ../pki/ca.crt, --etcd-cafile은 /pki/etcd/ca.crt 
- 나머지 certificate와 key는 모두 pki 아래에 존재
- 즉, etcd의 ca.crt만 pki/etcd 아래에 존재하고 나머지는 모두 pki에 존재

![alt text](image-144.png)

- etcd는 모두 pki/etcd 아래에 존재

![alt text](image-145.png)

## Certificates API

- Certificate 관리법과 Kubernetes 내 Certificate API가 무엇인지

#### 이전

- CA Server와 다양한 구성 요소에 대한 Certificate 설정
- 내가 cluster의 유일한 관리자이자 사용자
  - Admin Certificate와 key 가짐

#### Certificates API

| 새로운 사용자이자 관리자 등장

- 이때 새로운 Admin이 Cluster에 접속해야해 Certificate 한 쌍과 Cluster에 액세스할 key 한 쌍 필요
  - private key를 만들어 Certificate Sign Request를 생성해 나에게 전송
  - 내가 유일한 관리자이므로 Certificate Sign Request를 내 CA Server로 가져가 CA Server의 private key와 root Certificate를 사용해 서명
 - 내가 인증서를 생성해 새로운 사람에게 전달
 - 그 사람은 유효한 인증서와 key 쌍을 가지게 되어 cluster에 접근 가능
 - BUT Certificate는 만료 기한이 있고 만료되면 새 csr을 생성해 CA에 서명을 받는 과정을 다시 거쳐야 함

- `CA Server`
  - CA는 우리가 생성한 key와 certificate 파일 한 쌍
  - 이 파일 쌍에 접근할 수 있는 사람은 누구든 kubernetes 환경에 대한 certificate에 서명할 수 있음
  - 따라서 이 파일은 안전하게 보호되고 저장되어야 함
  - 완전히 안전한 서버에 설치한다고 가정하면, 그 서버는 나의 CA Server가 됨
    - Certificate와 key 파일은 해당 서버에 안전하게 저장
  - Certificate에 서명하고 싶을 때마다 해당 서버에 로그인해야 함
  - 현재 Certificate는 Master node에 존재하며, master node는 CA Server이기도 함
    - kubeadm도 key pair 파일 생성하고 master node에 저장

- 현재까지는 수동으로 요청에 서명했지만, **certificate sign request를 자동화할** 필요 있음
  - 인증서 만료 시에도 유용
- Kubernetes에는 `built-in Certificates API`가 있어 자동화 가능
  1. Create CertificateSigningRequest Object 
  - Certificate Sign Request를 Certificates API 호출을 통해 Kubernetes로 전송
  - Admin이 Certificate Sign Request를 받을 때 Master node에 로그인해 스스로 certificate에 서명하는 대신 `CertificateSigningRequest Object 생성`

  2. Review Requests
  - Object가 생성되면 모든 certificate sign request는 cluster admin이 보게 됨

  3. Approve Requests
  - request를 kubectl 유틸리티로 쉽게 검토하고 승인 가능

  4. Share Certs to Users
  - Certificate는 추출되어 사용자와 공유

- 실제 동작 과정
  1. 사용자가 private key 생성
  - `openssl genrsa -out jane.key 2048`
  2. key를 사용해 csr 생성해 Admin에게 요청 전송
  - `openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr`
  3. Admin이 key를 받아 CertificateSigningRequest Object 생성
  - 해당 object는 다른 kubernetes object와 마찬가지로 manifest 파일을 이용해 생성
  - YAML 파일
    - `request`: 사용자가 보낸 certificate signing request를 명시
      - plain text로 명시하지 않고 base64 명령을 이용해 암호화되어야 함
        - `cat jane.csr | base64` 의 결과(=인코딩된 텍스트)를 request 필드에 작성
    
    `jane-csr.yaml`
    ```
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: jane
    spec:
      expirationSeconds: 600  #seconds
      usages:
      - digital signature
      - key encipherment
      - server auth
      request:
        [base64로 인코딩된 값]
    ```

  4. object가 생성되면 조회 가능
  - `kubectl get csr`

  5. 요청 승인
  - `kubectl certificate approve jane`
  - 이 certificate는 추출되어 사용자와 공유될 수 있음

- `kubectl get csr jane -o yaml`의 certificate를 디코딩하려면 `echo "[인코딩값] | base64 --decode"`
- Controlplane에서 certificate 관련 작업은 `Controller Manager에 의해 실행`됨
  - Controller Manager 내에는 `CSR-APPROVING`과 `CSR-SIGNING` 존재

- 누구든 Certificate에 서명해야 한다면 서버의 root Certificate와 private key 필요
  - `cat /etc/kubernetes/manifests/kube-controller-manager.yaml`에 ca.crt와 ca.key 존재

## Practice Test - Certificates API

Q2

`cat [CSR NAME] | base64 -w 0`

- `-w 0`을 안 하면 한 줄이 아니게 되어 오류 발생 => 반드시 작성해야 함
- base64로 인코딩한 값을 YAML 파일에 작성

Q4

`kubectl certificate approve [CSR NAME]`

- 승인시켜줘야 함

Q7

`k get csr agent-smith -o yaml > smith.yaml`

- csr의 group을 보려면 describe가 아닌 yaml 파일로 확인해야 함

Q8

`kubectl certificate deny [CSR NAME]`

- csr 거절

## KubeConfig

## Practice Test - KubeConfig

## API Groups

## Authorization

## Role Based Access Controls

## Practice Test - RBAC

## Cluster Roles and Role Bindings

## Practice Test - Cluster Roles and Role Bindings

-65
--0805

## Service Accounts

## Practice Test - Service Accounts

## Image Security

## Practice Test - Image Security

## Pre-requisite - Security in Docker

## Security Context

## Practice Test - Security Contexts

## Network Policy

## Developing network policies

## Practice Test - Network Policy

-115
--0806
