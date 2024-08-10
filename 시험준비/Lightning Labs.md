| 틀린 문제: 2,4,6

#### Q1
```
Upgrade the current version of kubernetes from 1.29.0 to 1.30.0 exactly using the kubeadm utility. Make sure that the upgrade is carried out one node at a time starting with the controlplane node. To minimize downtime, the deployment gold-nginx should be rescheduled on an alternate node before upgrading each node.
```
#### A1

1. Master node 업그레이드

    1. "new package repositories hosted at pkgs.k8s.io" 링크에서 최신 package repository 가져오기
        - **이때 원하는 버전으로 버전 변경** 

    2. kubeadm upgrade
        - **첫번째 단계에서 버전 변경**
         그대로 따라하기

    3. Drain the node
        - 버전에서 x 빼기

    4. Upgrade kubelet and kubectl

    5. Uncordon

- 여기까지 완료해야 버전 업그레이드 완료

2. Worker node

    - `ssh node01`

    1. "new package repositories hosted at pkgs.k8s.io" 링크에서 최신 package repository 가져오기
        - **이때 원하는 버전으로 버전 변경** 

    2. kubeadm upgrade

    3. `exit` 후 controlplane에서 Node Drain

    4. `ssh node01`로 node01 들어가서 kubelet and kubectl Upgrade

    5. `exit` 후 controlplane에서 Uncordon


#### Q2 => 다시 풀 것
```
Print the names of all deployments in the admin2406 namespace in the following format:

DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE

<deployment name>   <container image used>   <ready replica count>   <Namespace>
. The data should be sorted by the increasing order of the deployment name.


Example:

DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE
deploy0   nginx:alpine   1   admin2406
Write the result to the file /opt/admin2406_data.
```
#### A2

`kubectl -n admin2406 get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin2406_data`


#### Q3
```
 A kubeconfig file called admin.kubeconfig has been created in /root/CKA. There is something wrong with the configuration. Troubleshoot and fix it.
```

#### A3

- `cp /root/CKA/admin.kubeconfig $HOME/.kube/config`
- kubectl 명령어 시 오류 발생 => controlplane port 문제
- controlplane port는 6443 

#### Q4
```
Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.
```

#### A4 => set image 할 때 `--record` 작성 잊지 말 것

- `k create deploy nginx-deploy --image nginx:1.16 --replicas 1`
- `kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record`
    - 이때 `record` 반드시 작성할것 !! docs에는 없음 
- 확인: `kubectl rollout status deployment/nginx-deploy`

#### Q5

```
A new deployment called alpha-mysql has been deployed in the alpha namespace. However, the pods are not running. Troubleshoot and fix the issue. The deployment should make use of the persistent volume alpha-pv to be mounted at /var/lib/mysql and should use the environment variable MYSQL_ALLOW_EMPTY_PASSWORD=1 to make use of an empty root password.


Important: Do not alter the persistent volume.
```

#### A5

- PVC Definition file
    - accessModes 추가
    - resources.requests.storage 1로 감소
    - storageClassName 변경
    - volumeName 추가

- Deployment Definition file
    - PVC 이름 변경

#### Q6 => 다시 풀기

```
Take the backup of ETCD at the location /opt/etcd-backup.db on the controlplane node.
```

#### A6

```
export ETCDCTL_API=3
etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/etcd-backup.db
```

- 이때 앞의 --cacert가 `/pki/etcd/ca.crt`, --cert가 `/pki/etcd/server.crt`, --key가 `/pki/etcd/server.key`

#### Q7

```
Create a pod called secret-1401 in the admin1401 namespace using the busybox image. The container within the pod should be called secret-admin and should sleep for 4800 seconds.

The container should mount a read-only secret volume called secret-volume at the path /etc/secret-volume. The secret being mounted has already been created for you and is called dotfile-secret.
```

#### A7

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: secret-1401
  name: secret-1401
  namespace: admin1401

spec:
  containers:
  - args:
    - dry-run=client
    image: busybox
    imagePullPolicy: Always
    name: secret-admin
    resources: {}
    command: ["/bin/sh"]
    args: ["-c", "sleep 4800"]
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-t4cnz
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  - name: kube-api-access-t4cnz
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```