Q5. ClusterIP Service 생성

```
Create a service messaging-service to expose the messaging application within the cluster on port 6379.

Use imperative commands.
```

A5

- ClusterIP의 경우 `k expose` 명령어 사용해 생성

Q7. Static Pod 생성

```
Create a static pod named static-busybox on the controlplane node that uses the busybox image and the command sleep 1000.
```

A7

1. 원하는 node에 접근
- `ssh node01`

2. kubelet configuration에서 static pod 경로 확인
- `cat /var/lib/kubelet/config.yaml`
    - (결과) staticPodPath: /etc/kubernetes/manifests

- 문서
    - Static pod 검색 > Create static Pods > Set Kubelet Parameters Via A Configuration File > Configuring each kubelet in your cluster using kubeadm(여기서 kubelet config yaml 파일 경로 찾기)

3. 해당 경로에 원하는 리소스 생성
- `/etc/kubernetes/manifests/static-web.yaml` 추가 또는 변경
- /etc/kubernetes/manifests 아래에 static pod 존재
- 또는 `kubectl run nginx-static-pod --image=nginx --port=80 --dry-run=client -o yaml > nginx-static-pod.yaml`로 한번에 생성

Q10. NodePort Service 생성

```
Expose the hr-web-app as service hr-web-app-service application on port 30082 on the nodes on the cluster.

The web application listens on port 8080.
```

A10

- `k create svc nodeport` 명령어 사용
- pod의 label과 service의 selector 맞추기

Q11. 결과 파일에 작성

```
Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt.

The osImages are under the nodeInfo section under status of each node.
```


A11

- `kubectl get nodes -o custom-columns=OS-IMAGE:.status.nodeInfo.osImage > /opt/outputs/nodes_os_x43kj56.txt`

- Command
    - `k get nodes --help` 에서 custom-columns 문서 찾기

Q12. PV 생성

```
Create a Persistent Volume with the given specification: -

Volume name: pv-analytics

Storage: 100Mi

Access mode: ReadWriteMany

Host path: /pv/data-analytics
```

A12

- 문서
    - PV 검색 > Persistent Volume >  Example of hostPath type Volume