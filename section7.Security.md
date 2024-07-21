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
    - 파일은 3개의 열과 password, username, user id로 구성
        - CSV 파일에서 선택적으로 4번째 열에 group을 지정해 할당할 수 있음
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


=> 웹 서버에서는 TLS 클라이언트 인증서가 구현되지 않고, 호스트 아래에 구현되어 있음
    - 일반 사용자는 인증서를 수동으로 생성하고 관리할 필요 없음

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



## TLS in Kubernetes

## TLS in Kubernetes - Certificate Creation

## View Certificate Details

