# k8s 1.29.3 on docker

## 初始化

```bash
# 关闭防火墙 selinux
systemctl disable --now firewalld
setenforce 0
sed -i '/SELINUX=/s@enforcing@disabled@g' /etc/selinux/config

# 设置最大文件打开数 最大进程数
cat << EOF | tee -a /etc/security/limits.conf
* soft nofile 655350
* hard nofile 655350
* soft nproc  655350
* hard nproc  655350
EOF

# 开启IP转发 swap优化等
cat << EOF | tee /etc/sysctl.d/k8s.conf
wappiness = 5
vm.panic_on_oom = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_syncookies = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-arptables = 1
net.ipv4.ip_forward = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.netfilter.nf_conntrack_max = 2310720
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963
fs.nr_open=52706963
EOF
# 配置生效
sysctl -p /etc/sysctl.d/k8s.conf

```

## docker k8s yum源

```bash
cat << EOF | tee /etc/yum.repos.d/docker-ce.repo
[docker-ce-stable]
name=Docker CE Stable - \$basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/\$releasever/\$basearch/stable
enabled=1
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF

cat << EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.29/rpm/
enabled=1
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.29/rpm/repodata/repomd.xml.key
EOF

```

## docker 安装 和cri-docker
```bash
# 安装docker
yum -y install docker

# 创建配置
mkdir -p /etc/docker/

cat << EOF | tee /etc/docker/daemon.json
{
    "data-root": "/data/docker",
    "registry-mirrors": ["https://frz7i079.mirror.aliyuncs.com"],
    "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
# 重载配置
systemctl daemon-reload
# 启动
systemctl enable --now docker containerd

# 下载 cri-docker
curl -o  cri-dockerd-0.3.12-3.el7.x86_64.rpm  https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.12/cri-dockerd-0.3.12-3.el7.x86_64.rpm

# 安装
yum -y install ./cri-dockerd-0.3.12-3.el7.x86_64.rpm

# 配置
cat << EOF | tee /usr/lib/systemd/system/cri-docker.socket
[Unit]
Description=CRI Docker Socket for the API
PartOf=cri-docker.service

[Socket]
ListenStream=/var/run/cri-dockerd.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
EOF

cat << EOF | tee /usr/lib/systemd/system/cri-docker.service
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target
Requires=cri-docker.socket

[Service]
Type=notify

ExecStart=/usr/bin/cri-dockerd \
          --container-runtime-endpoint=unix:///var/run/cri-docker.sock \
          --network-plugin=cni \
          --cni-bin-dir=/opt/cni/bin \
          --cni-conf-dir=/etc/cni/net.d \
          --image-pull-progress-deadline=30s \
          --pod-infra-container-image=registry.k8s.io/pause:3.9 \
          --docker-endpoint=unix:///run/docker.sock \
          --cri-dockerd-root-directory=/data/docker
ExecReload=/bin/kill -s HUP \$MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target

EOF

# 重载配置
systemctl daemon-reload
# 启动
systemctl enable --now cri-docker

```

## 安装k8s

```bash
# 安装 1.29.3 版本
yum -y install kubeadm-1.29.3-150500.1.1 kubectl-1.29.3-150500.1.1 kubelet-1.29.3-150500.1.1
# 配置
cat << EOF | tee /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"
EOF
# 设置自启
systemctl enable kubelet

```

### 拉取国内镜像

```bash
# 源镜像
kubeadm config images list > source.images.list
# 阿里镜像
sed 's@registry.k8s.io@registry.cn-hangzhou.aliyuncs.com/google_containers@g' source.images.list > ali.images.list
sed -i '/dns/s@coredns/coredns@coredns@g' ali.images.list
# 拉取阿里镜像
awk '{print "docker pull " $1}' ali.images.list | bash
# 改名
paste ali.images.list source.images.list  |  awk '{print "docker  tag " $1 " " $2}' | bash
# 删除阿里镜像
awk '{print "docker rmi " $1}' ali.images.list  | bash
# 删除临时记录文件
rm -rf source.images.list ali.images.list

```

### 单节点初始化

```bash
# 初始化 注意网段别重复
kubeadm init --cri-socket  unix:///var/run/cri-docker.sock  --pod-network-cidr 10.1.0.0/16 --service-cidr 10.2.0.0/16
# 指定镜像仓库 初始化
kubeadm init --cri-socket  unix:///var/run/cri-docker.sock  --pod-network-cidr 10.1.0.0/16 --service-cidr 10.2.0.0/16  --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
# 获取权限
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
# 删除污点（根据需求，这里删除是为了在master上跑任务）
kubectl taint node --all node-role.kubernetes.io/control-plane-
```

### calico网络插件

```bash
# 下载网络插件
wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
# 安装网络插件
kubectl apply -f calico.yaml

```