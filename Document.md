# 首先建立三台虛擬機
```bash
$ vim create_vm_settings.sh
...
vmnames="k8s-master k8s-node1 k8s-node2"
...
sh create_vm_settings.sh
```

# Kubernetes
![alt K8S][id]

[id]: https://github.com/d93y70123123/Kubernetes/blob/master/kubernetes-logo.png "logo"  
Kubernetes（常簡稱為K8s），是用於自動部署、擴展和管理容器化（containerized）應用程式的開源系統。主要提供「跨主機集群的自動部署、擴展以及運行應用程式容器的平台」。  
# K8S的架構
![alt K8S-archtecture](https://github.com/d93y70123123/Kubernetes/blob/master/kubernetes-archtecture.jpg "archtecture")  
Master負責協調叢集，其中主要腳色:  
* API server : 所有的 K8s 操作都是透過 API Server。   
* etcd : 負責與 API Server 溝通。  
* k8s-schedule : 進行調度，分配最適合的 Node 來執行 Container。
* controller-manager : 負責所有的控制功能。  

Node負責執行應用程式，其中主要腳色:  
* kubelet : 安裝於每一個 Node 上，負責與 API Server 溝通。  
* kube-proxy : 提供給 kubectl 或 kubelet 進行 API Server 連線 。  
* container runtime : 就是容器啦，像是:docker。  

# 安裝K8S
### 安裝注意事項
* CPU至少兩個
* 記憶體至少2G
* SWAP必須禁用
* cluster內的網路必須接通
* 節點的MAC以及productID不要一樣 ( cat /sys/class/dmi/id/product_uuid )
***
1. 修改hostname

2. 將selinux設置成permissive  
  查看selinux狀態 : getenforce

3. 關閉swap
```
# vim /etc/fstab
...
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
(註解此行)
...

swapon -s
swapoff -a
```

4. 關閉防火牆
* firewalld跟iptables都關閉
  
* 如果有開防火牆的話，為了確保網路不會被iptables略過會加入下面兩行
```bash
vim /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1

sysctl --system
```

5. 新增容器庫
```bash
$ vim /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
```
6. yum update

7. 安裝 kubelet kubeadm kubectl
```bash
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

8. 啟動kubelet
```bash
systemctl start kubelet
systemctl enable kubelet
```
`這邊因為還沒初始化的關係，kubelet會啟動失敗`  

9. 增加自動補齊功能
```
$ echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc
```
10. 重開機吧 reboot

## 簡單介紹k8s的兩個主要指令
* kubeadm：建立節點必備，可以快速幫助你建立 Master 以及增加 Node
* kubectl：負責溝通整個叢集的工具，可以利用它管理以及建立 k8s 的最小單位 "pod"等功能。

# 建立叢集
## Master建立  
安裝docker 
```bash
wget https://download.docker.com/linux/centos/docker-ce.repo -P /etc/yum.repos.d/
yum install docker-ce docker-ce-cli containerd.io
systemctl start docker
systemctl enable docker
```
1. Master節點初始化
```
kubeadm init --pod-network-cidr=10.244.0.0/16
..
....
.....
初始化過程中若出現下面這段文字，代表初始化成功
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
2. 先記下要加入叢集的的金鑰
```
如果忘記，可以用下面的方法查
kubeadm token list
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```
3. 為Master節點建立幾個檔案
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
4. 為所有的POD建立可以互相溝通的網路
```
*這邊使用flannel的CNI
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml
```  
## Node建立
```diff
- 工作節點也需要安裝Ｄocker喔！！  
```

重複安裝的步驟，可以將下面的指令段落做成腳本執行  
`*每個節點都要做` 
```bash
#!/bin/bash

hostnamectl set-hostname [改節點的名稱]

# create k8s's repo
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

# Set SELinux in permissive mode
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# close swap
swapoff -a
sed -i 's/\/dev\/mapper\/centos-swap/#\/dev\/mapper\/centos-swap/g' /etc/fstab

# set bridge for k8s
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

yum update -y

# install k8s
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
```

## 用k8s製作容器
pod：最小單位，可用來創建和部署的單元。
pod可以單獨運行一個容器也可以運行多個容器，運行多個容器時共用 "IP" 以及 "儲存的資源"。  
![alt k8s](https://github.com/d93y70123123/Kubernetes/blob/master/module_03_nodes.svg "pods")  
pod的可以用指令建立或是yaml、json建立，這邊用yaml檔來建立pod：  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-kubernetes
  labels:
    app: webserver
spec:
  containers:
  - name: my-k8s
    image: nginx
```
1. 建立 pod
```bash
$ kubectl create -f myk8s.yaml
...建立成功後
pod/hello-kubernetes created
```
2. 確認 pod 有沒有活著
```bash
$ kubectl get pods

NAME               READY   STATUS    RESTARTS   AGE
hello-kubernetes   1/1     Running   0          32s
```
3. 讓外部與 pod 連線  
```
$ kubectl expose pods hello-kubernetes --type="NodePort" --port=80

service/hello-kubernetes exposed
```
4. 查看、指令對外開啟的 port
```
$ kubectl get service

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
hello-kubernetes   NodePort    10.102.55.116   <none>        80:32136/TCP   74s
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP        8d

$ kubectl edit service hello-kubernetes
... 接著會進入編輯模式，跟 vim 大同小異
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2019-06-09T16:26:55Z"
  labels:
    app: webserver
  name: hello-kubernetes
  namespace: default
  resourceVersion: "972631"
  selfLink: /api/v1/namespaces/default/services/hello-kubernetes
  uid: 65506bf5-8ad3-11e9-bdb0-14dae957dc6b
spec:
  clusterIP: 10.102.55.116
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: (30000-32767)
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: webserver
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```
* 在本機上要開啟port才能對外開放
`iptables -t nat -A PREROUTING -i eno1 -p tcp -m tcp --dport [port] -j DNAT --to-destination 192.168.19.X:[port]`
5. 接著就可以從外部連接上container  
在瀏覽器貼打上 > http://自己的IP:port

## 部屬環境的好幫手Deployment
![alt k8s-deployment](https://github.com/d93y70123123/Kubernetes/blob/master/deployment-rs-pod.svg "k8s-deployment")
#### k8s提供的deployment可以做到以下的事情:  
* 建立pod
* 升級到某個版本
* 回朔到某個版本  

#### deployment主要管理的兩個單位:
* pods：k8s的最小單位
* Replicaset：升級以及回朔的方便工具  
一樣用yaml建立Deployment：
```
$ vim my-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
1. 建立Deployment
```bash
$ kubectl create -f my-deployment.yaml --record

deployment.apps/my-deployment creat
```

2. 檢查Deployment是否建立成功
```bash
$ kubectl get deployments.apps,pods -o wide

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES        SELECTOR
deployment.apps/my-deployment   2/2     2            2           50s   nginx        nginx:1.7.9   app=nginx

NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
pod/hello-kubernetes                1/1     Running   0          24h   10.244.1.37   k8s1   <none>           <none>
pod/my-deployment-6dd86d77d-8n78n   1/1     Running   0          50s   10.244.1.44   k8s1   <none>           <none>
pod/my-deployment-6dd86d77d-xw8c5   1/1     Running   0          50s   10.244.1.43   k8s1   <none>           <none>
```
*被Deployment建立出來的pod的名字格式:[deploymentname]-[亂碼]  

3. 當負載不夠時，可以增加容器

4. 更新系統
```bash
$ kubectl set image deployment.apps/my-deployment nginx=nginx:1.17.0 --record

deployment.apps/my-deployment image updated

查看更新狀態
$ kubectl rollout status deployment my-deployment

Waiting for deployment "my-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
deployment "my-deployment" successfully rolled out
```

5. 系統回朔
當系統崩潰或是版本不穩定就可以用這個功能
假設系統更新到錯誤版本：
```diff
$ kubectl set image deployment.apps/my-deployment nginx=nginx:10.17.0 --record

$ kubectl rollout status deployment my-deployment

Waiting for deployment "my-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
(應該會一直卡在這裡...)

$ kubectl  get pods
NAME                             READY   STATUS             RESTARTS   AGE
hello-kubernetes                 1/1     Running            0          25h
- my-deployment-6f6986d7b6-nf24d   0/1     ImagePullBackOff   0          8m31s
my-deployment-9dd464697-5fcsb    1/1     Running            0          9m45s
my-deployment-9dd464697-lfl59    1/1     Running            0          10m
my-deployment-9dd464697-q2knp    1/1     Running            0          9m42s

$ kubectl describe deployments my-deployment
  查看詳細的資料

$ kubectl rollout history deployment
  可以看之前做的各種版本，加上 --revision=2可以看該版本更詳細的資訊
```
回朔到前個版本：
```
$ kubectl rollout undo deployment.v1.apps/nginx-deployment
deployment.extensions/my-deployment rolled back

$ kubectl rollout status deployment my-deployment
deployment "my-deployment" successfully rolled out
```
6. 擴展Deployment
```bash
$ kubectl scale deployment --replicas=10 my-deployment

$ kubectl get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
hello-kubernetes                1/1     Running   0          25h   10.244.1.37   k8s1   <none>           <none>
my-deployment-9dd464697-5fcsb   1/1     Running   0          20m   10.244.1.70   k8s1   <none>           <none>
my-deployment-9dd464697-9w4s6   1/1     Running   0          42s   10.244.1.74   k8s1   <none>           <none>
my-deployment-9dd464697-bvg89   1/1     Running   0          42s   10.244.1.79   k8s1   <none>           <none>
my-deployment-9dd464697-lfl59   1/1     Running   0          20m   10.244.1.69   k8s1   <none>           <none>
my-deployment-9dd464697-mmttv   1/1     Running   0          42s   10.244.1.77   k8s1   <none>           <none>
my-deployment-9dd464697-mqdgl   1/1     Running   0          42s   10.244.1.78   k8s1   <none>           <none>
my-deployment-9dd464697-nr7lm   1/1     Running   0          42s   10.244.1.73   k8s1   <none>           <none>
my-deployment-9dd464697-q2knp   1/1     Running   0          20m   10.244.1.71   k8s1   <none>           <none>
my-deployment-9dd464697-qgbm9   1/1     Running   0          42s   10.244.1.75   k8s1   <none>           <none>
my-deployment-9dd464697-s9lxk   1/1     Running   0          42s   10.244.1.76   k8s1   <none>           <none>
```

## 安裝 dashboard
dashboard 是k8s的附加元件，可以讓k8s在網頁上監控及管理。
1. 建立 dashboard 相關服務
```bash
$ kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/alternative/kubernetes-dashboard.yaml

serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
```
2. 看 dashboard 將port開在哪
```bash
$ kubectl get service -n kube-system

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
kube-dns               ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP,9153/TCP   8d
kubernetes-dashboard   NodePort    10.103.165.8   <none>        80:30773/TCP             4m2s
```
3. 若是出現權限問題，就幫他增加權限
```
# vim dashboard-admin.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
  
kubectl create -f dashboard.yaml
```
### 參考資料 ###
* K8S建立：https://kubernetes.io/docs/setup/independent/install-kubeadm/  

