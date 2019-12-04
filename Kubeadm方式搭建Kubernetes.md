## Kubernetes kubeadm 安装记录

- kubernetes yum 源

```bash
cat /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
```

- 遇到下面的错误，不用管，先不启动kubelet，init之后会有的

```
unable to load client CA file /etc/kubernetes/pki/ca.crt: open /etc/kubernetes/pki/ca.crt: no such file or directory
```

- 

```bash
kubeadm init \
   --kubernetes-version=v1.13.0 \
   --pod-network-cidr=10.244.0.0/16 \
   --apiserver-advertise-address=192.168.233.140 \
   --ignore-preflight-errors=Swap
```

- etcd启动遇到如下问题

```bash
etcdmain: open /etc/kubernetes/pki/etcd/peer.crt: permission denied

关闭selinux
[root@localhost ~]# setenforce 0 //临时关闭
[root@localhost ~]# getenforce
Permissive
[root@localhost ~]# vim /etc/sysconfig/selinux //永久关闭
# 将SELINUX=enforcing 改为 SELINUX=disabled 。
```

- kubeadm init如果遇到下面的问题

```bash
[ERROR KubeletVersion]: the kubelet version is higher than the control plane version. This is not a supported version skew and may lead to a malfunctional cluster
#---------
提高kubernetes的版本
```

- 重置 安装失败的kubeadm init

```
$ kubeadm reset
$ ifconfig cni0 down && ip link delete cni0
$ ifconfig flannel.1 down && ip link delete flannel.1
$ rm -rf /var/lib/cni/
```

- init之后如果kubectl get cs 返回如下错误

```bash
The connection to the server localhost:8080 was refused - did you specify the right host or port?
#或者下面的错误
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
#--------
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- 记住下面的命令

```bash
  kubeadm join 192.168.233.140:6443 --token u80nom.6cqe1vomk37a2use --discovery-token-ca-cert-hash sha256:ed1e75a3aacfa74d0afcdb0b0035227cf7b06f93e31292cac0121426f291c9e4

# -----------
#token忘记了怎么办
kubeadm token create
# sha256忘记了怎么办
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

- not ready describe后查看到下面错误

```bash
cni config uninitialized
# ----
# 找下面两个位置更改
vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
修改 /var/lib/kubelet/kubeadm-flags.env 文件删除 network的配置
KUBELET_KUBEADM_ARGS=--cgroup-driver=systemd --cni-bin-dir=/opt/cni/bin --cni-conf-dir=/etc/cni/net.d --network-plugin=cni
# 重启kubelet


#加载ipvs相关内核模块
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack_ipv4

#设置ipvs开机启动
cat <<EOF >>  /etc/rc.local
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack_ipv4
EOF

#配置kubelet使用国内镜像
DOCKER_CGROUPS=$(docker info | grep 'Cgroup' | cut -d' ' -f3)
echo $DOCKER_CGROUPS
cat <<EOF> /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=$DOCKER_CGROUPS --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
EOF

#启动kubelet
systemctl daemon-reload
systemctl enable kubelet && systemctl restart kubelet
```

- 创建nginx pod

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-controller
spec:
  replicas: 2
  selector:
    name: nginx
  template:
    metadata:
      labels:
        name: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
           - containerPort: 80
```

- 删除pod一定要删除rc，不需要删除pod，会自动删除，如果执行删除pod还会创建

```bash
kubectl delete rc nginx-controller
```

- node(s) had taints that the pod didn't tolerate

```bash
# kubernetes出于安全考虑默认情况下无法在master节点上部署pod
# kubectl taint nodes --all node-role.kubernetes.io/master-
```

- 让dashboard可以被外面访问

```bash
kubectl proxy --address=192.168.233.140 --disable-filter=true
# ip是master的ip
```

- 显示kube-system的pod

```bash
kubectl get pods --all-namespaces              
```

- describe kube-system的pod

```bash
kubectl describe pod -n kube-system etcd-master
kubectl --namespace kube-system logs kube-flannel-ds-amd64-c7rfz
```

- 授予Dashboard账户集群管理权限

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-admin
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard-admin
  namespace: kube-system
```

kubectl -n kube-system get secret | grep kubernetes-dashboard-admin

 kubectl describe -n kube-system secret/kubernetes-dashboard-admin-token-ddskx

- 修改dashboard 的type

```bash
kubectl patch svc kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}' -n kube-system
```

https://www.cnblogs.com/klvchen/p/9963642.html

https://blog.csdn.net/u012375924/article/details/78987263

- [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1

```bash
# 修改文件 /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
# 添加上面两句
sysctl -p /etc/sysctl.d/k8s.conf
# 修改 vim  /usr/lib/sysctl.d/00-system.conf
net.ipv4.ip_forward=1
#添加上面这句
systemctl restart network
```

注：这个错误路由异常问题，我是在reset之后出现的这个问题。我是上面两步都需要执行才好使

- 

<http://192.168.233.140:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login>