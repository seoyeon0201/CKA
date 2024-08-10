| 틀린 문제: 4, 6, 7, 8


Q1. ETCD Backup

```
Take a backup of the etcd cluster and save it to /opt/etcd-backup.db.
```

A1

- `ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd-backup.db`

- 문서
    - Certificate 검색 > PKI certificates and requirements | Kubernetes (Certificate 경로 찾기)
    - ETCD Backup 검색 > Operating etcd clusters for Kubernetes (ETCDCTL 명령어 찾기)
        - 이 명령어의 certificate와 key는 모두 .../pki/etcd 내에 존재
        - --cacert는 ca.crt, --cert는 server.crt, --key는 server.key

Q4

```
A pod definition file is created at /root/CKA/use-pv.yaml. Make use of this manifest file and mount the persistent volume called pv-1. Ensure the pod is running and the PV is bound.


mountPath: /data

persistentVolumeClaim Name: my-pvc

persistentVolume Claim configured correctly

pod using the correct mountPath

pod using the persistent volume claim?
```

A4

- 이름 잘못 설정
- 굳이 PVC에 PV 작성할 필요 없음

Q6. Certificate Signing Requests

```
Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr.


Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.

Please refer the documentation to see an example. The documentation tab is available at the top right of terminal.

CSR: john-developer Status:Approved

Role Name: developer, namespace: development, Resource: Pods

Access: User 'john' has appropriate permissions
```

A6

1. CSR 생성 

```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUt2Um1tQ0h2ZjBrTHNldlF3aWVKSzcrVVdRck04ZGtkdzkyYUJTdG1uUVNhMGFPCjV3c3cwbVZyNkNjcEJFRmVreHk5NUVydkgyTHhqQTNiSHVsTVVub2ZkUU9rbjYra1NNY2o3TzdWYlBld2k2OEIKa3JoM2prRFNuZGFvV1NPWXBKOFg1WUZ5c2ZvNUpxby82YU92czFGcEc3bm5SMG1JYWpySTlNVVFEdTVncGw4bgpjakY0TG4vQ3NEb3o3QXNadEgwcVpwc0dXYVpURTBKOWNrQmswZWhiV2tMeDJUK3pEYzlmaDVIMjZsSE4zbHM4CktiSlRuSnY3WDFsNndCeTN5WUFUSXRNclpUR28wZ2c1QS9uREZ4SXdHcXNlMTdLZDRaa1k3RDJIZ3R4UytkMEMKMTNBeHNVdzQyWVZ6ZzhkYXJzVGRMZzcxQ2NaanRxdS9YSmlyQmxVQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQ1VKTnNMelBKczB2czlGTTVpUzJ0akMyaVYvdXptcmwxTGNUTStsbXpSODNsS09uL0NoMTZlClNLNHplRlFtbGF0c0hCOGZBU2ZhQnRaOUJ2UnVlMUZnbHk1b2VuTk5LaW9FMnc3TUx1a0oyODBWRWFxUjN2SSsKNzRiNnduNkhYclJsYVhaM25VMTFQVTlsT3RBSGxQeDNYVWpCVk5QaGhlUlBmR3p3TTRselZuQW5mNm96bEtxSgpvT3RORStlZ2FYWDdvc3BvZmdWZWVqc25Yd0RjZ05pSFFTbDgzSkljUCtjOVBHMDJtNyt0NmpJU3VoRllTVjZtCmlqblNucHBKZWhFUGxPMkFNcmJzU0VpaFB1N294Wm9iZDFtdWF4bWtVa0NoSzZLeGV0RjVEdWhRMi80NEMvSDIKOWk1bnpMMlRST3RndGRJZjAveUF5N05COHlOY3FPR0QKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  usages:
  - digital signature
  - key encipherment
  - client auth
```

2. `k certificate approve [CSR]`
3. developer 이름의 role과 rolebinding 생성

Q7

A7

- service
    - `kubectl run test-nslookup --image=busybox:1.28 --restart=Never -it --rm -- nslookup nginx-resolver-service > /root/CKA/nginx.svc`

- pod
    - `kubectl run test-nslookup --image=busybox:1.28 --restart=Never -it --rm -- nslookup 10-244-192-2.default.pod.cluster.local > /root/CKA/nginx.pod`

- Pod는 IP를 -로 연결하고 default.pod.cluster.local과 같이 작성해야 하고 Service는 Service 이름으로 nslookup

Q8

```
Create a static pod on node01 called nginx-critical with image nginx and make sure that it is recreated/restarted automatically in case of a failure.

Use /etc/kubernetes/manifests as the Static Pod path for example.
```

A8

1. Controlplane에서 pod YAML file 생성

- `k run > static.yaml`

2. Pod YAML file을 Node01로 이동

- `scp static.yaml node01:/root/`

3. Node01 접근

- `ssh node01`

4. Node에 /etc/kubernetes/manifests 폴더 생성 

- `mkdir -p /etc/kubernetes/manifests`

5. Node의 /root 디렉토리에 존재하는 YAML 파일 이동

- `cp /root/static.yaml /etc/kubernetes/manifests`