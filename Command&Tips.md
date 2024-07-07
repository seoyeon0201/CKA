## Certification Tips

#### Options

`--dry-run=client`
    - 리소스를 생성하지 않고 리소스를 생성할 수 있는지 여부만 단순히 테스트

`-o yaml`
    - YAML 형식의 리소스 정의 파일 출력

#### Imperative Commands

| Pod 생성은 run, ReplicaSet이나 Deploy, Namespace는 create

| NodePort 생성 시에는 nodePort 포트 번호를 작성해야 하기 때문에 `kubectl create` 명령어가 유리하고, ClusterIP Service 생성 시에는 Selector가 원하는 pod와 일치해야 하기 때문에 `kubectl expose` 명령어가 유리

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

| 아래에서 Service 생성 방법으로 `k expose`와 `k create`가 나오는데, `k expose` 권장

`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

- redis-service 이름의 ClusterIP Service 생성하고 6379 포트를 가지는 redis 이름의 pod에 노출
    - 이때 pod의 label을 자동으로 Service의 Selector로 동작

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml`

- 이 Service의 selector는 app=redis가 되므로 pod에 또다른 label이 존재한다면 **Service의 selector 수정 필요**

`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`

- nginx-service 이름의 service 생성. pod의 label이 자동으로 Service의 Selector로 들어감
    - 단, **NodePort 포트 번호를 설정할 수 없으므로 definition file 생성 후 수정 필요**

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`

- **pod의 label이 Service의 Selector로 들어가지 않아 수정 필요**


`kubectl replace --force -f [YAML]`

- YAML 파일 수정 사항 있는 경우, 이전의 리소스를 삭제하고 다시 생성해야 하는데 replace 명령어가 이를 한 번에 진행시켜줌
    - 이때 반드시 `--force`가 정상적으로 동작



## 명령어

- 복사 붙여 넣기는 `ctrl + shift + c` , `ctrl + shift + v`

#### 1. 리소스 생성 & 조회

`kubectl run [POD NAME] --image=[IMAGE NAME]`

- pod만 생성할 때 빠르게 생성

`kubectl run [POD NAME] --image=[IMAGE NAME] --dry-run=client -o yaml > redis.yaml`

- `--dry-run=client`
- `-o yaml > redis.yaml`으로 yaml 파일 생성 가능

`kubectl get pods -o wide`

- pod node, IP 조회 가능

`cat pod.yaml`

- yaml 파일 조회

`kubectl describe pod [POD NAME] | grep [보고 싶은 단어]`

- 원하는 정보 빠르게 찾을 수 있음
- `-B 5` 해당 단어가 나온 줄로부터 5줄 뒤까지 조회 가능

#### 2. ReplicaSet 

| replicaset = rs

| Replicaset 변경할 때에는 (1) pod를 모두 지우거나 (2) replicaset yaml 파일을 delete 하는 작업 필요

| Replicaset replicas 개수 변경
1. `k scale --replicas=[NUMBER] -f [REPLICASET YAML FILE]`
2. `k scale rs [REPLICASET NAME] --replicas=[NUMBER]`
3. 또는 yaml 파일 변경 후 `k edit -f [YAML FILE]` 
4. 또는 `k edit replicaset [REPLICASET NAME]`으로 정의 파일 들어가서 replicas 변경

`kubectl explain replicaset`

- replicaset의 kind, version, field 등 나타남

#### 3. Deployment

| deployment = deploy

`kubectl get all`

- pods, replicaset, deployment 모두 조회 가능

`kubectl create deploy --help`

- 명령어로 deployment 생성 시 사용하는 명령어 조회 가능

#### 4. Service

`k create service --help`

- service 생성 명령어를 모르는 경우

`kubectl run [POD NAME] --image=[IMAGE NAME] --port=[PORT NUMBER] --expose=true`

- 특정 image와 port 번호를 가지는 Pod와 그 pod의 label을 가지는 Service가 한 번에 생성

| NodePort는 `kubectl create` 명령어로 nodePort 번호 지정하고, ClusterIP는 `kubectl expose` 명령어로 pod의 label을 selector로 가지는 것이 유리

`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

- redis-service 이름의 ClusterIP Service 생성하고 6379 포트를 가지는 redis 이름의 pod에 노출
    - 이때 pod의 label을 자동으로 Service의 Selector로 동작

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml`

- 이 Service의 selector는 app=redis가 되므로 pod에 또다른 label이 존재한다면 **Service의 selector 수정 필요**

`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`

- nginx-service 이름의 service 생성. pod의 label이 자동으로 Service의 Selector로 들어감
    - 단, **NodePort 포트 번호를 설정할 수 없으므로 definition file 생성 후 수정 필요**

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`

- **pod의 label이 Service의 Selector로 들어가지 않아 수정 필요**


#### 5. Namespace

| namespace = ns

`kubectl config set-context ${kubectl config current-context} --namespace=dev`

- namespace 변경

`kubectl get pods --all-namespaces` 또는 `kube get pods -A`

- 모든 namespace에서 리소스 조회

#### 6. Scheduling (Node)

| Pod를 특정 Node에 위치시키고 싶은 경우, YAML 파일에 nodeName 필드 추가 이후 아래 명령어 수행

1. `kubectl replace --force -f [YAML]`
    - 이전의 리소스를 제거하고 즉시 재생성

2. 또는 `kubectl delete -f [YAML]`한 후 `kubectl apply -f [YAML]`
    - 이전의 리소스를 제거한 후 생성 명령어 진행

`kubectl get pods --watch` 

- pod의 상태 변화 모니터링 가능

`kubectl get pods -o wide` 

- 더 많은 정보 조회 가능. 특히 Node 조회 가능

#### 7. Labels and Selectors

`k get pods --selector env=dev`

- label이 env=dev인 pod 탐색

`k get pods --selector env=dev --no-headers | wc -l`

- `wc -l`은 숫자 빠르게 계산해주는 명령어(wc -l: word cound와 line의 축약어). 맨 윗 줄(NAME|READY|STATUS|RESTARTS|AGE)이 출력되지 않도록 `--no-headers` 옵션 사용

`kubectl get all --selector env=prod,bu=finance,tier=frontend`

- 여러 조건을 모두 만족하는 리소스 찾기

#### 8. Taints And Tolerations

`kubectl taint node [NODE NAME] [KEY]=[VALUE]:[EFFECT]`

- Taint 명령어. (Tolerations는 Pod의 Defintion file에 작성)

`kubectl taint node [NODE NAME] [해당 NODE의 TAINTS]-`
    - Ex. `kubectl taint node controlplane node-role.kubernetes.io/master:NoSchedule-`

- 마지막에 `-`를 붙여 Node의 Taint 제거

#### 9. Node Selector & Node Affinity

`kubectl label nodes [NODE NAME] [LABEL KEY]=[LABEL VALUE]`

- Node에 크기 관련한 label 붙임 (Pod의 Definition file에 nodeSelector 작성)

`kubectl get node [NODE NAME] --show-labels`

- Node가 가지는 label 확인

#### 10. Resource Requirements and Limits

`k edit` 명령어 사용해 requests와 limits를 변경할 수 없지만, 이때 새로운 파일에 변경 사항이 적용되어 `k replace --force -f` 명령어로 변경사항 저장 가능

#### 11. DaemonSets

`kubectl create deployment [DAEMONSET NAME] -n [NAMESPACE] --image=[IMAGE] --dry-run=client -o yaml > daemonset.yaml`

- Daemonset은 replicaset, deployment와 유사한 형태이기 때문에 --dry-run=client -o yaml을 사용해 생성
- replicas와 strategy 필드 삭제


#### 12. Static Pods


`kubectl get pod [POD NAME] -o yaml`

- YAML 파일의 ownerReferences의 kind가 Node인 경우 Static pod
- Static pod는 이름 뒤에 node명이 붙음

`cat /var/lib/kubelet/config.yaml`

- 위 YAML 파일에서 staticPodPath가 static pod의 definition file이 존재하는 경로

`k run [POD NAME] --image [IMAGE NAME] --dry-run=client -o yaml>[YAML FILE NAME]`
`cp [이동시킬 YAML 파일] [이동할 경로]`

- YAML 파일 생성 후 경로 이동

`kubectl get nodes -o wide`
`ssh [NODE INTERNAL IP]`
- 해당 node IP를 확인한 후 이동

(node로 이동 후) `cat /var/lib/kubelet/config.yaml`

- 디렉토리 경로 확인

`cd [디렉토리 경로]`
(디렉토리 경로로 접근 후) `rm [FILE NAME]`

(node 탈출) `exit`

#### 13. Logging&Monitoring

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

| Application 로그 조회 

`kubectl logs -f [POD NAME]`

- 실시간으로 pod의 container의 로그 조회 가능

`kubectl logs -f [POD NAME] [CONTAINER NAME]`

- 실시간으로 해당 container 로그 조회 가능