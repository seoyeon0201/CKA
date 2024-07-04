## Certification Tips

`kubectl run nginx --image=nginx`

- 이름이 nginx이고 이미지로 nginx 사용하는 pod 생성

`kubectl run nginx --image=nginx --dry-run=client -o yaml`

- pod의 YAML 파일 생성(-o yaml)하고 실제로 생성하지는 않음(--dry-run=client. 예상 결과만 알려줌)

`kubectl create deployment --image=nginx nginx`

- deployment 생성

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

- deployment YAML 파일 생성하고 실제로 생성하지는 않음

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml`

- nginx-deployment.yaml 이름으로 deployment YAML 파일 생성

`kubectl create -f nginx-deployment.yaml`

- 생성한 YAML 파일로 리소스 생성

`kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`

- nginx-deployment.yaml이라는 deployment 정의 파일 생성. image와 replicas 설정 가능










## 명령어

- 복사 붙여 넣기는 `ctrl + shift + c` , `ctrl + shift + v`

#### 리소스 생성

`kubectl run [POD NAME] --image=[IMAGE NAME]`

- pod만 생성할 때 빠르게 생성

`kubectl run [POD NAME] --image=[IMAGE NAME] --dry-run=client -o yaml > redis.yaml`

- `--dry-run=client`
- `-o yaml > redis.yaml`으로 yaml 파일 생성 가능

#### 리소스 조회

`kubectl get pods -o wide`

- pod node, IP 조회 가능

`cat pod.yaml`

- yaml 파일 조회

`kubectl describe pod [POD NAME] | grep [보고 싶은 단어]`

- 원하는 정보 빠르게 찾을 수 있음
- `-B 5` 해당 단어가 나온 줄로부터 5줄 뒤까지 조회 가능

#### ReplicaSet 

| replicaset = rs

| Replicaset 변경할 때에는 (1) pod를 모두 지우거나 (2) replicaset yaml 파일을 delete 하는 작업 필요

| Replicaset replicas 개수 변경
1. `k scale --replicas=[NUMBER] -f [REPLICASET YAML FILE]`
2. `k scale rs [REPLICASET NAME] --replicas=[NUMBER]`
3. 또는 yaml 파일 변경 후 `k edit -f [YAML FILE]` 
4. 또는 `k edit replicaset [REPLICASET NAME]`으로 정의 파일 들어가서 replicas 변경

`kubectl explain replicaset`

- replicaset의 kind, version, field 등 나타남

#### Deployment

`kubectl get all`

- pods, replicaset, deployment 모두 조회 가능

`kubectl create deploy --help`

- 명령어로 deployment 생성 시 사용하는 명령어 조회 가능

