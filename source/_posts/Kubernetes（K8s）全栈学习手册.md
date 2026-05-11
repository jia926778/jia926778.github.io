---
title: Kubernetes（K8s）学习手册
date: 2026-04-30 19:30:00
tags: [K8s, 技术分享, Kubernetes]
categories: 工具使用
toc: true
---
# Kubernetes（K8s）全栈学习手册

## 文档核心说明

本文档适用于 K8s 入门学习者、运维开发工程师、容器化应用落地从业者，严格遵循「从基础到进阶、从理论到实操、从核心到周边」的学习逻辑，完整覆盖 K8s 全链路知识体系，所有操作步骤、参数标准、技术结论均 100% 可复现，无冗余重复内容。
本文档核心知识模块包括：

1. K8s 基础架构与核心概念

2. 本地开发到生产级集群的全场景搭建方案

3. kubectl 命令行工具全高频用法

4. K8s 全资源 YAML 规范与全参数详解

5. 核心资源实战配置与落地操作

6. 高频故障排查思路与解决方案

7. 生产环境落地最佳实践与避坑指南

## 一、K8s 基础认知

### 1.1 K8s 核心定义与价值

Kubernetes（简称 K8s）是谷歌开源的**容器编排引擎**，用于自动化部署、扩缩容、管理容器化应用，解决单机 Docker 的核心痛点：

- 跨主机容器调度与集群化管理

- 故障自愈（容器崩溃自动重建）

- 服务发现与负载均衡

- 零停机滚动更新 / 版本回滚

- 配置与敏感信息统一管理

- 资源精细化管控与自动扩缩容

### 1.2 K8s 集群核心架构与组件

K8s 集群分为 \\*\\* 控制平面（Master 节点）**和**工作节点（Worker/Node 节点）\\*\\* 两大核心部分，所有组件通过`kube-apiserver`进行通信。

|节点类型|核心组件|核心功能|
|---|---|---|
|控制平面|kube-apiserver|集群唯一入口，提供 RESTful API，所有组件交互的中枢，负责认证、授权、准入控制|
|控制平面|etcd|集群的分布式键值数据库，存储集群所有状态、配置数据，是集群的 “大脑”|
|控制平面|kube-scheduler|负责 Pod 调度，根据资源、亲和性、污点等规则，为 Pod 选择最合适的 Node 节点|
|控制平面|kube-controller-manager|运行各类控制器：副本控制器、节点控制器、端点控制器等，保障集群状态符合预期|
|控制平面|cloud-controller-manager|可选组件，对接云厂商基础设施，实现负载均衡、云主机等资源的适配|
|工作节点|kubelet|节点核心代理，与 Master 通信，管理本机 Pod / 容器的生命周期，确保容器按预期运行|
|工作节点|kube-proxy|维护节点网络规则，实现 Service 的四层负载均衡，保障集群内外网络通信|
|工作节点|容器运行时|实际运行容器的引擎，**推荐 containerd**（K8s 1.24 \+ 已弃用 Dockershim），支持所有符合 CRI 标准的运行时|

### 1.3 K8s 核心资源概念速览

|资源类型|核心作用|
|---|---|
|Namespace|命名空间，实现集群资源的逻辑隔离，区分环境、团队|
|Pod|K8s 最小调度 / 部署单元，一个 Pod 包含 1 个或多个紧密耦合的容器，共享网络、存储|
|Deployment|无状态应用的核心控制器，管理 Pod 的副本数、滚动更新、回滚，是生产中最常用的资源|
|Service|服务发现与负载均衡，为 Pod 提供固定访问地址，屏蔽 Pod 的动态变化|
|ConfigMap|存储非敏感配置信息，支持环境变量、文件挂载，实现配置与镜像解耦|
|Secret|存储敏感信息（密码、证书、密钥），加密存储，避免敏感信息明文泄露|
|PV/PVC|持久化存储，PV 是集群存储资源，PVC 是用户对存储的申请，实现存储与应用生命周期解耦|
|StatefulSet|有状态应用控制器（数据库、中间件），提供稳定的网络标识、持久化存储、有序部署 / 更新|
|DaemonSet|守护进程集，在每个（或指定）Node 上运行一个 Pod 副本，适合日志采集、监控 Agent 等场景|
|Ingress|集群七层流量入口，实现域名路由、SSL 终止、路径转发，突破 NodePort 的端口限制|
|RBAC|基于角色的权限控制，定义用户 / 服务账户对集群资源的操作权限，保障集群安全|
|HPA|水平自动扩缩容，基于 CPU、内存等指标自动调整 Pod 副本数，适配业务流量波动|

## 二、K8s 集群环境搭建

本章节覆盖从本地开发测试到生产级集群的全场景搭建方案，所有步骤可直接复现，适配 K8s 1.30 \+ 稳定版。

### 2.1 本地开发环境（Minikube，新手首选）

Minikube 是单节点 K8s 集群，一键启动，适合本地学习、开发测试，支持 Windows/Mac/Linux 全平台。

#### 前置条件

- 2 核 2G 以上配置，20G 以上磁盘空间

- 已安装容器运行时（Docker/containerd）

- 开启 CPU 虚拟化支持

#### 安装与启动步骤

1. 安装 Minikube

```bash
# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Mac（Homebrew）
brew install minikube

# Windows（Chocolatey）
choco install minikube
```

2. 启动集群（国内用户需指定镜像源）

```bash
# 基础启动（默认使用docker驱动）
minikube start --image-mirror-country=cn --kubernetes-version=stable

# 自定义配置（指定CPU、内存、版本）
minikube start --cpus=2 --memory=4096 --image-mirror-country=cn --kubernetes-version=v1.30.0
```

3. 验证集群状态

```bash
# 查看集群状态
minikube status

# 查看节点信息
kubectl get nodes

# 进入集群可视化控制面板
minikube dashboard
```

### 2.2 本地多节点测试环境（Kind）

Kind（Kubernetes IN Docker）通过 Docker 容器模拟 K8s 节点，快速搭建多节点集群，适合本地测试多节点调度、高可用场景。

#### 安装与启动步骤

1. 安装 Kind

```bash
# Linux/Mac
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Mac Homebrew
brew install kind
```

2. 创建多节点集群配置文件`kind-cluster.yaml`

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

3. 创建集群

```bash
kind create cluster --config=kind-cluster.yaml --name=k8s-cluster
```

4. 验证集群

```bash
kubectl get nodes
```

### 2.3 生产级集群搭建（Kubeadm）

Kubeadm 是 K8s 官方推荐的集群搭建工具，可快速搭建符合生产标准的集群，支持单 Master 和高可用架构。

#### 前置条件

- 服务器配置：Master 节点 2 核 4G 以上，Worker 节点 2 核 2G 以上，至少 1 台 Master\+1 台 Worker

- 操作系统：Ubuntu 22.04/24.04、CentOS Stream 9、Rocky Linux 9（推荐 Ubuntu）

- 网络要求：所有节点内网互通，主机名 / MAC 地址唯一，开放 6443、2379-2380、10250、10259 等端口

- 所有节点必须完成通用环境配置、容器运行时安装、K8s 核心组件安装

#### 步骤 1：所有节点通用环境配置

```bash
# 1. 关闭防火墙
sudo systemctl stop firewalld && sudo systemctl disable firewalld

# 2. 关闭SELinux（CentOS/Rocky）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config

# 3. 关闭Swap（K8s强制要求，必须永久关闭）
sudo swapoff -a
sudo sed -i '/swap/s/^/#/' /etc/fstab

# 4. 设置主机名（Master/Worker分别设置，确保唯一）
# Master节点执行
sudo hostnamectl set-hostname k8s-master
# Worker1节点执行
sudo hostnamectl set-hostname k8s-node1
# Worker2节点执行
sudo hostnamectl set-hostname k8s-node2

# 5. 配置hosts解析（所有节点执行，替换为实际服务器IP）
cat >> /etc/hosts <<EOF
192.168.1.100 k8s-master
192.168.1.101 k8s-node1
192.168.1.102 k8s-node2
EOF

# 6. 加载容器运行时所需内核模块
cat > /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay && sudo modprobe br_netfilter

# 7. 配置内核网络参数
cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```

#### 步骤 2：所有节点安装容器运行时 containerd

```bash
# 1. 安装containerd
sudo apt update && sudo apt install -y containerd.io

# 2. 生成默认配置文件
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# 3. 配置systemd cgroup驱动（K8s推荐）
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# 4. 配置国内镜像加速（可选，解决镜像拉取慢）
sudo sed -i 's#registry.k8s.io#registry.aliyuncs.com/google_containers#g' /etc/containerd/config.toml

# 5. 重启containerd并设置开机自启
sudo systemctl restart containerd
sudo systemctl enable containerd
```

#### 步骤 3：所有节点安装 K8s 核心组件

```bash
# 1. 安装依赖
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl gpg

# 2. 添加K8s官方GPG密钥（国内可使用阿里云镜像）
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes-apt-keyring.gpg

# 3. 添加K8s软件源
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# 4. 安装kubeadm、kubelet、kubectl，锁定版本避免自动更新
sudo apt update && sudo apt install -y kubeadm=1.30.0-1.1 kubelet=1.30.0-1.1 kubectl=1.30.0-1.1
sudo apt-mark hold kubeadm kubelet kubectl

# 5. 验证安装
kubeadm version
kubectl version --client
```

#### 步骤 4：Master 节点初始化控制平面

**仅在 Master 节点执行**，初始化集群控制平面，生成证书、静态 Pod 配置、Node 加入命令。

```bash
# 初始化命令（替换为实际Master节点IP，国内指定阿里云镜像源）
sudo kubeadm init \
  --apiserver-advertise-address=192.168.1.100 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.30.0 \
  --pod-network-cidr=10.244.0.0/16 \
  --service-cidr=10.96.0.0/12 \
  --cri-socket /run/containerd/containerd.sock
```

初始化成功后，**务必保存输出的****`kubeadm join`****命令**，后续 Worker 节点加入集群需要使用。

#### 步骤 5：配置 kubectl 权限（Master 节点执行）

```bash
# 为普通用户配置kubeconfig文件
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 验证集群状态
kubectl get nodes
```

#### 步骤 6：安装 CNI 网络插件（必装，Master 节点执行）

K8s 集群必须安装 CNI 网络插件才能实现 Pod 间通信，否则 CoreDNS 会处于 Pending 状态，**推荐 Calico**。

```bash
# 安装Calico网络插件
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# 验证网络插件安装成功（等待所有Pod变为Running状态）
kubectl get pods -n kube-system -w
```

#### 步骤 7：Worker 节点加入集群

**仅在所有 Worker 节点执行**，使用初始化时输出的`kubeadm join`命令，示例如下：

```bash
# 替换为你自己的token和hash值
sudo kubeadm join 192.168.1.100:6443 \
  --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
  --cri-socket /run/containerd/containerd.sock
```

> 若 token 过期，可在 Master 节点执行`kubeadm token create --print-join-command`重新生成加入命令。
>
>

#### 步骤 8：集群最终验证（Master 节点执行）

```bash
# 查看所有节点状态，全部变为Ready即为成功
kubectl get nodes

# 查看所有系统组件Pod状态，全部Running即为集群正常
kubectl get pods -n kube-system
```

## 三、kubectl 核心命令行工具

kubectl 是 K8s 集群的命令行管理工具，所有集群操作都通过 kubectl 完成，核心命令结构为：

```bash
kubectl [command] [资源类型] [资源名称] [flags]
```

### 3.1 kubectl 基础配置

- 集群内：kubeadm 安装时已同步安装

- 本地远程管理：下载对应版本 kubectl，将 Master 节点的`\~/.kube/config`文件复制到本地`\~/.kube/config`，即可远程管理集群

### 3.2 高频核心命令分类详解

|命令分类|核心命令|核心作用|高频示例|
|---|---|---|---|
|集群管理|cluster-info|查看集群核心组件地址与状态|`kubectl cluster-info`|
|集群管理|top|查看节点 / Pod 资源使用率|`kubectl top nodes` / `kubectl top pods -n default`|
|集群管理|api-resources|查看集群支持的所有资源类型|`kubectl api-resources`|
|集群管理|config|管理 kubeconfig 配置与上下文|`kubectl config get-contexts` / `kubectl config use-context xxx`|
|集群管理|version|查看客户端和服务端版本|`kubectl version --output=yaml`|
|资源查看|get|获取资源列表，最常用基础命令|`kubectl get pods` / `kubectl get deployments -o wide`|
|资源查看|describe|查看资源详细信息与事件（排障核心）|`kubectl describe pod nginx-xxx-xxx`|
|资源查看|logs|查看 Pod 内容器日志（排障核心）|`kubectl logs -f nginx-xxx-xxx` / `kubectl logs nginx-xxx-xxx -c 容器名`|
|资源查看|exec|进入 Pod 内容器执行命令|`kubectl exec -it nginx-xxx-xxx -- /bin/bash`|
|资源操作|apply|声明式创建 / 更新资源（生产推荐）|`kubectl apply -f nginx-deploy.yaml`|
|资源操作|create|命令式创建资源|`kubectl create namespace dev`|
|资源操作|delete|删除资源|`kubectl delete pod nginx-xxx-xxx` / `kubectl delete -f nginx-deploy.yaml`|
|资源操作|edit|在线编辑资源配置|`kubectl edit deployment nginx`|
|资源操作|patch|在线补丁更新资源|`kubectl patch deployment nginx -p \&\#39;\{\&\#34;spec\&\#34;:\{\&\#34;replicas\&\#34;:5\}\}\&\#39;`|
|应用运维|rollout|管理应用滚动更新 / 回滚|`kubectl rollout status deployment nginx` / `kubectl rollout undo deployment nginx`|
|应用运维|scale|手动扩缩容副本数|`kubectl scale deployment nginx --replicas=5`|
|应用运维|label|为资源添加 / 修改标签|`kubectl label pods nginx-xxx-xxx env=dev`|
|应用运维|cp|在本地与 Pod 之间复制文件|`kubectl cp nginx-xxx-xxx:/etc/nginx/nginx.conf ./nginx.conf`|

## 四、K8s 全资源 YAML 规范与参数详解

K8s 资源推荐使用**YAML 文件声明式管理**，符合基础设施即代码（IaC）理念，便于版本控制、复用和回滚。本章节覆盖所有核心资源的 YAML 规范、全字段详解，严格区分必填 / 可选参数，标注生产注意事项。

### 4.1 YAML 通用顶层结构

**所有 K8s 资源 YAML 都必须遵循这个顶层框架**，仅`spec`内部配置随资源类型变化：

```yaml
# 顶级字段，所有资源必含
apiVersion: <API组/版本>  # 必填，使用的K8s API版本
kind: <资源类型>          # 必填，定义要创建的资源对象类型
metadata:                 # 必填，元数据，定义资源的唯一标识、分类等信息
spec:                     # 必填，核心配置，定义资源的期望状态、行为规则
status:                   # 系统自动生成，用户禁止手动配置，集群维护的资源实际状态
```

|顶级字段|字段类型|核心说明|常用取值示例|
|---|---|---|---|
|`apiVersion`|字符串|必填，指定操作资源的 API 版本，不同资源对应不同 API 组|核心资源：`v1`<br>工作负载：`apps/v1`<br>Ingress：`networking.k8s.io/v1`|
|`kind`|字符串|必填，定义资源类型，大小写敏感，必须与 apiVersion 匹配|`Pod`、`Deployment`、`Service`|
|`metadata`|对象|必填，资源的元数据，所有资源的元数据结构完全通用|-|
|`spec`|对象|必填，资源的核心配置，定义用户期望的资源状态|-|
|`status`|对象|系统只读，由 K8s 控制器实时更新，用户手动配置无效|如 Pod 的`status.phase`、Deployment 的`status.readyReplicas`|

### 4.2 metadata 元数据全字段详解（所有资源通用）

`metadata`定义资源的唯一标识、分类、附加信息，是所有资源的通用配置。

|字段|字段类型|可配置性|核心说明与注意事项|
|---|---|---|---|
|`name`|字符串|必填|资源名称，**同一命名空间内必须唯一**，仅支持小写字母、数字、`-`、`_`、`.`，最长 253 字符，创建后不可修改|
|`namespace`|字符串|可选|资源所属命名空间，不填默认`default`；**集群级资源（Node、PV、ClusterRole）禁止填写**|
|`labels`|键值对|可选|资源标签，用于筛选、关联、调度，核心作用是与`selector`匹配，实现资源关联|
|`annotations`|键值对|可选|非标识性元数据，用于存储附加信息，**不用于资源筛选 / 匹配**|
|`ownerReferences`|对象数组|可选|所有者引用，定义资源的父子关系，实现级联删除，用户一般无需手动配置|
|`finalizers`|字符串数组|可选|终结器，用于资源删除前的钩子逻辑，防止资源误删|
|`uid`/`resourceVersion`/`creationTimestamp`|-|系统只读|集群自动生成，用户不可修改，用于全局唯一标识、乐观锁并发控制|

### 4.3 核心资源 spec 字段全参数详解

#### 4.3.1 Pod 资源（最小调度单元）

Pod 是 K8s 最小的部署 / 调度单元，**生产环境不建议直接创建 Pod**，而是通过 Deployment/StatefulSet 等控制器管理；所有工作负载的`spec.template`都是 Pod 模板，结构与 Pod 完全一致。
**apiVersion: v1**
**kind: Pod**

##### Pod.spec 核心字段

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`containers`|对象数组|必填|Pod 内的容器列表，至少配置 1 个容器|
|`initContainers`|对象数组|可选|初始化容器，在业务容器启动前按顺序执行，全部成功才启动业务容器，结构与`containers`完全一致|
|`volumes`|对象数组|可选|Pod 内的存储卷定义，供容器挂载使用|
|`restartPolicy`|字符串|可选|容器重启策略，可选值：`Always`（默认）、`OnFailure`、`Never`；Job/CronJob 推荐用`OnFailure/Never`|
|`nodeSelector`|键值对|可选|节点选择器，将 Pod 调度到匹配对应标签的节点|
|`affinity`|对象|可选|亲和性 / 反亲和性，比 nodeSelector 更灵活的调度规则，包含节点亲和性、Pod 亲和性、Pod 反亲和性（生产高可用必配）|
|`tolerations`|对象数组|可选|容忍度，匹配节点的污点，让 Pod 可以调度到有对应污点的节点|
|`hostNetwork`|布尔值|可选|是否使用主机网络，默认 false；开启后 Pod 直接使用节点网络栈，端口不能冲突，生产慎用|
|`dnsPolicy`|字符串|可选|DNS 策略，可选值：`ClusterFirst`（默认）、`Default`、`ClusterFirstWithHostNet`、`None`|
|`serviceAccountName`|字符串|可选|Pod 使用的 ServiceAccount 名称，定义 Pod 访问 K8s API 的权限|
|`automountServiceAccountToken`|布尔值|可选|是否自动挂载 ServiceAccount 的 token，默认 true；不需要访问 API 的 Pod 建议设为 false|
|`terminationGracePeriodSeconds`|整数|可选|优雅终止宽限期，默认 30 秒；容器收到终止信号后，最多等待多久执行收尾，超时后强制终止|
|`securityContext`|对象|可选|Pod 级安全上下文，对所有容器生效，控制运行用户、权限等安全配置|
|`imagePullSecrets`|对象数组|可选|拉取私有镜像的 Secret 名称列表|

##### containers 容器核心字段

每个容器必须配置`name`和`image`，其余字段按需配置：

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`name`|字符串|必填|容器名称，**同一个 Pod 内必须唯一**|
|`image`|字符串|必填|容器镜像地址；生产禁止用`latest`标签，必须固定版本|
|`imagePullPolicy`|字符串|可选|镜像拉取策略，可选值：`IfNotPresent`（默认）、`Always`、`Never`|
|`command`|字符串数组|可选|容器启动命令，**覆盖镜像的 ENTRYPOINT**|
|`args`|字符串数组|可选|容器启动参数，**覆盖镜像的 CMD**|
|`ports`|对象数组|可选|容器暴露的端口列表|
|`ports\[\].containerPort`|整数|必填|容器内监听的端口，范围 1-65535|
|`env`|对象数组|可选|环境变量列表，支持直接赋值和 ConfigMap/Secret 引用|
|`resources`|对象|生产必配|容器的资源请求与限制，避免资源争抢和节点 OOM|
|`resources.requests`|对象|可选|资源请求，调度器的调度依据；CPU 单位`m`（1 核 = 1000m），内存单位`Mi/Gi`|
|`resources.limits`|对象|可选|资源上限，容器使用超过限制会被 OOM 终止或 CPU 限流|
|`livenessProbe`|对象|生产必配|存活探针，检测容器是否存活，连续失败会重启容器|
|`readinessProbe`|对象|生产必配|就绪探针，检测容器是否可接收流量，连续失败会从 Service 端点中移除|
|`startupProbe`|对象|可选|启动探针，针对慢启动应用，启动成功后才执行存活 / 就绪探针|
|`volumeMounts`|对象数组|可选|将 Pod 内的卷挂载到容器内|
|`volumeMounts\[\].name`|字符串|必填|卷名称，必须与`spec.volumes`中的 name 完全一致|
|`volumeMounts\[\].mountPath`|字符串|必填|容器内的挂载路径|
|`volumeMounts\[\].subPath`|字符串|可选|挂载卷内的子路径，避免挂载覆盖目录内原有文件|
|`lifecycle`|对象|可选|容器生命周期钩子，`postStart`（启动后执行）、`preStop`（停止前执行，生产必配）|
|`securityContext`|对象|可选|容器级安全上下文，覆盖 Pod 级配置|

##### 探针通用配置

`livenessProbe`/`readinessProbe`/`startupProbe`结构完全一致，通用参数如下：

|字段|类型|默认值|说明|
|---|---|---|---|
|`initialDelaySeconds`|整数|0|首次探测的延迟时间，适配应用启动耗时|
|`periodSeconds`|整数|10|探测的间隔时间|
|`timeoutSeconds`|整数|1|单次探测的超时时间|
|`failureThreshold`|整数|3|连续失败多少次视为异常|

支持 3 种探测方式（三选一）：`exec`（执行命令）、`httpGet`（HTTP 请求）、`tcpSocket`（TCP 端口探测）。

#### 4.3.2 Deployment 资源（无状态应用核心控制器）

Deployment 是生产环境无状态应用的首选控制器，管理 ReplicaSet，实现 Pod 的副本管理、滚动更新、版本回滚、自愈能力。
**apiVersion: apps/v1**
**kind: Deployment**

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`replicas`|整数|可选|期望的 Pod 副本数，默认 1|
|`selector`|对象|必填|标签选择器，用于匹配管理的 Pod，**必须与 spec.template.metadata.labels 完全匹配**，创建后不可修改|
|`selector.matchLabels`|键值对|二选一|直接匹配 Pod 的标签，简单场景使用|
|`selector.matchExpressions`|对象数组|二选一|表达式匹配，支持`In`、`NotIn`、`Exists`、`DoesNotExist`|
|`template`|对象|必填|Pod 模板，结构与 Pod 资源完全一致，定义要创建的 Pod 的完整配置|
|`strategy`|对象|可选|Pod 更新策略，默认`RollingUpdate`|
|`strategy.type`|字符串|可选|更新类型，可选值：`RollingUpdate`（默认，滚动更新，零停机）、`Recreate`（重建更新，会停机）|
|`strategy.rollingUpdate`|对象|可选|滚动更新配置，type 为 RollingUpdate 时生效|
|`rollingUpdate.maxSurge`|整数 / 字符串|可选|滚动更新时，最多超出期望副本数的数量 / 比例，默认 25%；生产高可用建议设为 1|
|`rollingUpdate.maxUnavailable`|整数 / 字符串|可选|滚动更新时，最多不可用的 Pod 数量 / 比例，默认 25%；生产零停机建议设为 0|
|`revisionHistoryLimit`|整数|可选|保留的历史版本数，默认 10，用于版本回滚；设为 0 则无法回滚|
|`minReadySeconds`|整数|可选|Pod 就绪后，等待多久才认为 Pod 可用，默认 0；生产建议设为 3-5 秒|
|`paused`|布尔值|可选|是否暂停部署，默认 false；设为 true 后，修改 Pod 模板不会触发滚动更新|

#### 4.3.3 Service 资源（服务发现与负载均衡）

Service 为一组 Pod 提供固定访问地址，屏蔽 Pod 的动态 IP 变化，实现负载均衡，是 K8s 服务发现的核心。
**apiVersion: v1**
**kind: Service**

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`selector`|键值对|可选|标签选择器，匹配后端 Pod 的标签；无 selector 的 Service 用于映射外部服务或 Headless Service|
|`type`|字符串|可选|Service 类型，可选值：`ClusterIP`（默认）、`NodePort`、`LoadBalancer`、`ExternalName`|
|`ports`|对象数组|必填|端口映射列表，多端口时每个端口必须配置唯一 name|
|`ports\[\].port`|整数|必填|Service 自身暴露的端口，集群内访问的端口|
|`ports\[\].targetPort`|整数 / 字符串|必填|转发到 Pod 的端口，可以是端口号或 Pod 中定义的端口名称|
|`ports\[\].nodePort`|整数|可选|节点端口，type 为 NodePort/LoadBalancer 时生效，范围 30000-32767|
|`clusterIP`|字符串|可选|集群 IP，默认自动分配；设为`None`则为 Headless Service|
|`sessionAffinity`|字符串|可选|会话亲和性，可选值：`None`（默认）、`ClientIP`|
|`externalTrafficPolicy`|字符串|可选|外部流量策略，可选值：`Cluster`（默认）、`Local`（保留客户端源 IP）|
|`externalName`|字符串|必填|type 为 ExternalName 时必填，映射的外部域名|

#### 4.3.4 ConfigMap \&amp; Secret 配置管理资源

##### ConfigMap（非敏感配置）

用于存储非敏感配置信息，实现配置与镜像解耦，支持环境变量注入、文件挂载。
**apiVersion: v1**
**kind: ConfigMap**

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`data`|键值对|可选|存储非敏感配置，值可以是单行字符串或多行文本|
|`binaryData`|键值对|可选|存储二进制数据，值必须是 base64 编码|
|`immutable`|布尔值|可选|是否不可变，默认 false；设为 true 后禁止修改，提升集群性能|

##### Secret（敏感信息管理）

用于存储密码、证书、API 密钥、镜像仓库凭证等敏感信息，默认 base64 编码，支持加密存储。
**apiVersion: v1**
**kind: Secret**

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`type`|字符串|必填|Secret 类型，默认`Opaque`；常用类型：`kubernetes.io/dockerconfigjson`（镜像仓库凭证）、`kubernetes.io/tls`（TLS 证书）|
|`data`|键值对|可选|敏感数据，值必须是 base64 编码的字符串|
|`stringData`|键值对|可选|明文字符串，创建时自动编码为 base64 存入`data`，无需手动编码，推荐使用|
|`immutable`|布尔值|可选|是否不可变，默认 false，生产建议开启|

#### 4.3.5 StatefulSet 资源（有状态应用控制器）

用于部署有状态应用（MySQL、Redis、Kafka 等），提供稳定的网络标识、持久化存储、有序部署 / 更新。
**apiVersion: apps/v1**
**kind: StatefulSet**

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`serviceName`|字符串|必填|绑定的 Headless Service 名称，必须提前创建，为 Pod 提供稳定的 DNS 域名|
|`replicas`/`selector`/`template`|-|与 Deployment 完全一致|-|
|`volumeClaimTemplates`|对象数组|可选|PVC 模板，为每个 Pod 自动创建独立的 PVC，Pod 删除后 PVC 默认保留|
|`updateStrategy`|对象|可选|更新策略，默认`RollingUpdate`|
|`updateStrategy.type`|字符串|可选|更新类型：`RollingUpdate`（默认）、`OnDelete`（手动删除 Pod 才更新）|
|`updateStrategy.rollingUpdate.partition`|整数|可选|分区更新，仅更新序号≥partition 的 Pod，用于灰度发布|
|`podManagementPolicy`|字符串|可选|Pod 管理策略，可选值：`OrderedReady`（默认，有序部署）、`Parallel`（并行部署）|
|`persistentVolumeClaimRetentionPolicy`|对象|可选|K8s 1.27 \+ 支持，PVC 保留策略，定义 StatefulSet 删除 / 缩容时 PVC 的处理方式|

#### 4.3.6 Ingress 资源（七层流量入口）

实现集群七层流量管理，支持域名路由、路径转发、SSL 终止、限流等功能，必须先安装 Ingress Controller 才能使用。
**apiVersion: \[networking.k8s.io/v1\]\(networking.k8s.io/v1\)**
**kind: Ingress**

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`ingressClassName`|字符串|必填|Ingress 类名称，指定使用的 Ingress Controller|
|`rules`|对象数组|必填|路由规则列表|
|`rules\[\].host`|字符串|可选|域名，不填则为默认规则，匹配所有请求|
|`rules\[\].http.paths`|对象数组|必填|路径转发规则列表|
|`paths\[\].path`|字符串|必填|匹配路径|
|`paths\[\].pathType`|字符串|必填|路径类型，可选值：`Exact`（精确匹配）、`Prefix`（前缀匹配）、`ImplementationSpecific`|
|`paths\[\].backend.service.name`|字符串|必填|目标 Service 名称|
|`paths\[\].backend.service.port.number`|整数|必填|目标 Service 端口号|
|`tls`|对象数组|可选|HTTPS 配置|
|`tls\[\].hosts`|字符串数组|必填|域名列表，必须与证书中的域名匹配|
|`tls\[\].secretName`|字符串|必填|存储 TLS 证书的 Secret 名称|
|`defaultBackend`|对象|可选|默认后端，未匹配到任何规则的请求转发到这里|

#### 4.3.7 PV \&amp; PVC 持久化存储资源

- **PV（PersistentVolume）**：集群中的存储资源，由管理员创建，对接后端存储

- **PVC（PersistentVolumeClaim）**：用户对存储的申请，K8s 自动匹配符合条件的 PV，实现存储与应用解耦

##### PV 核心字段

**apiVersion: v1**
**kind: PersistentVolume**
（集群级资源，metadata 中禁止填写 namespace）

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`capacity`|对象|必填|存储容量，必须配置`storage`字段|
|`accessModes`|字符串数组|必填|访问模式，可选值：`ReadWriteOnce（RWO）`、`ReadOnlyMany（ROX）`、`ReadWriteMany（RWX）`、`ReadWriteOncePod（RWOP）`|
|`persistentVolumeReclaimPolicy`|字符串|必填|回收策略，可选值：`Retain`（保留，默认）、`Delete`（删除）|
|`storageClassName`|字符串|可选|存储类名称，用于 PVC 匹配|
|`volumeMode`|字符串|可选|卷模式，可选`Filesystem`（默认）、`Block`（块设备）|
|`nodeAffinity`|对象|可选|节点亲和性，定义哪些节点可以访问这个 PV|
|后端存储类型|对象|必填|只能选一种，常用：`nfs`、`csi`（生产推荐）、`local`等|

##### PVC 核心字段

**apiVersion: v1**
**kind: PersistentVolumeClaim**

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`accessModes`|字符串数组|必填|访问模式，需与 PV 匹配|
|`resources`|对象|必填|资源请求，必须配置`requests.storage`|
|`storageClassName`|字符串|可选|存储类名称，匹配对应 PV|
|`volumeName`|字符串|可选|直接绑定指定的 PV 名称|
|`selector`|对象|可选|标签选择器，筛选符合条件的 PV|

#### 4.3.8 RBAC 权限控制资源

RBAC 基于角色的访问控制，由 4 个核心资源组成，所有资源 apiVersion 均为`rbac.authorization.k8s.io/v1`。

##### Role \&amp; ClusterRole

- **Role**：命名空间级角色，定义对当前命名空间资源的操作权限

- **ClusterRole**：集群级角色，定义集群级资源、所有命名空间资源的操作权限

核心字段（两者结构完全一致）：

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`rules`|对象数组|必填|权限规则列表|
|`rules\[\].apiGroups`|字符串数组|必填|API 组，`\&\#34;\&\#34;`代表核心 v1 组|
|`rules\[\].resources`|字符串数组|必填|资源类型，如`\[\&\#34;pods\&\#34;, \&\#34;deployments\&\#34;\]`|
|`rules\[\].verbs`|字符串数组|必填|操作动词，常用：`get`、`list`、`watch`、`create`、`update`、`delete`|
|`rules\[\].resourceNames`|字符串数组|可选|限制仅能操作指定名称的资源|

##### RoleBinding \&amp; ClusterRoleBinding

- **RoleBinding**：命名空间级绑定，将 Role/ClusterRole 绑定到主体，仅对当前命名空间生效

- **ClusterRoleBinding**：集群级绑定，将 ClusterRole 绑定到主体，对整个集群生效

核心字段（两者结构完全一致）：

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`subjects`|对象数组|必填|授权主体列表，类型可选`User`、`Group`、`ServiceAccount`|
|`roleRef`|对象|必填|角色引用，创建后不可修改，固定 apiGroup 为`rbac.authorization.k8s.io`|

#### 4.3.9 HPA 水平自动扩缩容资源

基于 CPU、内存、自定义指标自动调整 Pod 副本数，适配业务流量波动。
**apiVersion: autoscaling/v2**
**kind: HorizontalPodAutoscaler**

|字段|字段类型|可配置性|核心说明|
|---|---|---|---|
|`scaleTargetRef`|对象|必填|扩缩容目标对象，需指定 apiVersion、kind、name|
|`minReplicas`|整数|可选|最小副本数，默认 1|
|`maxReplicas`|整数|必填|最大副本数，必须大于 minReplicas|
|`metrics`|对象数组|可选|扩缩容指标列表，类型可选`Resource`（CPU / 内存）、`Pods`、`Object`、`External`|
|`behavior`|对象|可选|扩缩容行为控制，避免频繁扩缩容抖动|
|`behavior.scaleUp.stabilizationWindowSeconds`|整数|可选|扩容稳定窗口，默认 0|
|`behavior.scaleDown.stabilizationWindowSeconds`|整数|可选|缩容稳定窗口，默认 300 秒|

## 五、核心资源实战操作

本章节提供所有核心资源的完整可运行 YAML 示例与操作命令，与前文参数详解一一对应，可直接复制执行。

### 5.1 命名空间（Namespace）管理

1. 创建 Namespace 配置文件`namespace-dev.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    env: dev
```

2. 核心操作命令

```bash
# 创建/更新Namespace
kubectl apply -f namespace-dev.yaml

# 查看所有命名空间
kubectl get namespaces

# 切换默认命名空间
kubectl config set-context --current --namespace=dev
```

### 5.2 无状态应用 Deployment 部署实战

1. 完整 Deployment 配置文件`nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: default
  labels:
    app: nginx
    env: prod
spec:
  replicas: 3
  revisionHistoryLimit: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
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
        image: nginx:1.27-alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 2
          periodSeconds: 5
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
```

2. 核心操作命令

```bash
# 创建/更新Deployment
kubectl apply -f nginx-deployment.yaml

# 查看Deployment状态
kubectl get deployments
kubectl describe deployment nginx-deploy

# 查看滚动更新状态
kubectl rollout status deployment nginx-deploy

# 版本回滚
kubectl rollout undo deployment nginx-deploy

# 手动扩缩容
kubectl scale deployment nginx-deploy --replicas=5
```

### 5.3 Service 服务发现配置实战

1. ClusterIP Service 配置文件`nginx-service-clusterip.yaml`（集群内访问）

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

2. NodePort Service 配置文件`nginx-service-nodeport.yaml`（对外暴露）

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-svc
  namespace: default
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

3. 核心操作命令

```bash
# 创建Service
kubectl apply -f nginx-service-clusterip.yaml

# 查看Service详情
kubectl get svc
kubectl describe svc nginx-svc
```

### 5.4 配置与密钥管理实战

1. ConfigMap 配置文件`nginx-configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
data:
  nginx.conf: |
    user nginx;
    worker_processes auto;
    events {
      worker_connections 1024;
    }
    http {
      server {
        listen 80;
        server_name localhost;
        location / {
          root /usr/share/nginx/html;
          index index.html index.htm;
        }
      }
    }
  env: prod
  log_level: info
```

2. 镜像仓库 Secret 创建命令

```bash
kubectl create secret docker-registry harbor-secret \
  --docker-server=harbor.example.com \
  --docker-username=admin \
  --docker-password=Harbor12345 \
  --docker-email=admin@example.com
```

3. ConfigMap 挂载到 Deployment 使用（仅展示核心配置）

```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.27-alpine
    env:
    - name: ENV
      valueFrom:
        configMapKeyRef:
          name: nginx-config
          key: env
    volumeMounts:
    - name: nginx-config-volume
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf
  volumes:
  - name: nginx-config-volume
    configMap:
      name: nginx-config
  imagePullSecrets:
  - name: harbor-secret
```

### 5.5 持久化存储 PV/PVC 实战

1. PV 配置文件`pv-nfs.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs-01
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  nfs:
    path: /data/nfs/pv01
    server: 192.168.1.100
```

2. PVC 配置文件`pvc-nfs.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs-01
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs
```

3. PVC 挂载到 Deployment 使用（核心配置）

```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.27-alpine
    volumeMounts:
    - name: data-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: pvc-nfs-01
```

### 5.6 有状态应用 StatefulSet 部署实战

```yaml
# Headless Service
apiVersion: v1
kind: Service
metadata:
  name: redis-headless
  namespace: default
spec:
  selector:
    app: redis
  clusterIP: None
  ports:
  - port: 6379
    targetPort: 6379
---
# StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: default
spec:
  serviceName: redis-headless
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7.2-alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "nfs"
      resources:
        requests:
          storage: 5Gi
```

### 5.7 Ingress 七层路由配置实战

1. 安装 Nginx Ingress Controller（国内镜像源）

```bash
kubectl apply -f https://gitee.com/mirrors/ingress-nginx/raw/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml
```

2. Ingress 配置文件`ingress-nginx.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: nginx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
  tls:
  - hosts:
    - nginx.example.com
    secretName: nginx-tls-secret
```

### 5.8 HPA 自动扩缩容配置实战

1. 前置条件：安装 metrics-server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

2. HPA 配置文件`hpa-nginx.yaml`

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deploy
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
```

## 六、常见故障排查与运维指南

### 6.1 故障排查核心思路

遵循**从宏观到微观、从集群到应用**的排查路径：

1. 检查集群节点状态：`kubectl get nodes`，确认所有节点 Ready

2. 检查资源事件：`kubectl describe 资源类型 资源名称`，查看 Events 事件（排障核心）

3. 检查 Pod 状态：`kubectl get pods`，确认 Pod 状态，定位异常类型

4. 检查容器日志：`kubectl logs -f Pod名称`，查看应用报错信息

5. 进入容器排查：`kubectl exec -it Pod名称 -- /bin/bash`，检查配置、网络、依赖等

### 6.2 高频故障场景与解决方案

#### 场景 1：Pod 一直处于 Pending 状态

**核心原因**：调度失败，没有节点能满足 Pod 的运行要求
**排查与解决**：

1. 执行`kubectl describe pod Pod名称`查看 Events 事件

2. 节点资源不足：节点 CPU / 内存耗尽，扩容节点或调整 Pod 的 resources.requests

3. 污点与容忍度不匹配：节点有污点，Pod 未配置对应的容忍度，添加容忍度或去除节点污点

4. PVC 绑定失败：PVC 不存在、未绑定 PV、存储类异常，检查 PVC/PV 状态

5. 节点亲和性不满足：Pod 配置的亲和性规则无匹配节点，调整亲和性规则

#### 场景 2：Pod 处于 ImagePullBackOff/ErrImagePull 状态

**核心原因**：镜像拉取失败
**排查与解决**：

1. 检查镜像地址是否正确，是否存在拼写错误

2. 检查镜像仓库是否可访问，节点网络是否正常

3. 私有镜像仓库：检查是否配置了 imagePullSecrets，Secret 是否正确

4. 国内网络问题：替换为国内镜像源，或配置镜像加速

#### 场景 3：Pod 处于 CrashLoopBackOff 状态

**核心原因**：容器启动成功后异常退出，反复重启
**排查与解决**：

1. 查看容器退出码：`kubectl describe pod Pod名称`，查看 State 中的 ExitCode

    - 退出码 0：正常退出，检查容器启动命令是否为前台运行

    - 退出码 1：应用通用错误，查看应用日志

    - 退出码 137：被 OOM 终止，内存超出 limits 限制，调大内存限制或优化应用内存

    - 退出码 139：段错误，应用程序崩溃

2. 查看应用日志：`kubectl logs Pod名称 --previous`（查看上一次崩溃的日志）

3. 检查健康检查配置：livenessProbe 配置错误，导致 K8s 不断重启 Pod

4. 检查配置文件 / 依赖：配置错误、依赖缺失、权限不足导致应用启动失败

#### 场景 4：Pod 处于 ContainerCreating 状态

**核心原因**：容器创建过程中失败
**排查与解决**：

1. 查看 Events 事件，常见原因：

    - 存储卷挂载失败：PVC 不存在、NFS 服务器不可访问、存储权限不足

    - ConfigMap/Secret 不存在：引用的配置 / 密钥未创建，检查名称和命名空间

    - CNI 网络插件异常：节点网络插件故障，重启节点 kubelet/containerd

    - 容器运行时异常：containerd 服务异常，重启 containerd 服务

#### 场景 5：Service 无法访问

**排查与解决**：

1. 检查 Service 的 selector 是否与 Pod 的 labels 完全匹配

2. 检查 Service 的 port/targetPort 是否正确，Pod 是否监听了对应的端口

3. 集群内直接访问 Pod IP \+ 端口，确认应用本身可正常访问

4. 检查 Pod 是否处于 Ready 状态，readinessProbe 是否通过

5. 检查节点防火墙 / 安全组是否放通了相关端口

## 七、生产环境最佳实践

### 7.1 集群安全加固

1. **最小权限原则**：通过 RBAC 为每个服务账户、用户分配最小必要权限，禁止使用 cluster-admin 权限运行普通应用

2. **Pod 安全加固**：禁止容器以 root 用户运行，配置 securityContext.runAsNonRoot=true；禁止特权容器，关闭 allowPrivilegeEscalation

3. **API Server 安全**：关闭匿名访问，开启 TLS 双向认证，限制 API Server 访问源 IP

4. **etcd 安全**：开启 etcd 数据加密，限制 etcd 访问仅允许控制平面节点访问，定期备份 etcd 数据

5. **镜像安全**：使用私有镜像仓库，对镜像进行安全扫描，禁止使用 latest 标签，固定镜像版本

6. **网络隔离**：通过 NetworkPolicy 实现 Pod 间、命名空间间的网络访问控制，默认拒绝非必要访问

### 7.2 资源管理与稳定性保障

1. **强制资源限制**：为所有 Pod 配置 resources.requests 和 limits，避免资源争抢和节点 OOM

2. **命名空间资源管控**：通过 LimitRange 设置命名空间默认资源限制，通过 ResourceQuota 限制命名空间总资源

3. **健康检查全覆盖**：为所有业务容器配置 livenessProbe、readinessProbe、startupProbe，精准感知应用健康状态

4. **优雅停机配置**：配置 preStop 钩子，处理应用关闭前的收尾工作，设置合理的 terminationGracePeriodSeconds，避免流量丢失

5. **高可用部署**：控制平面采用 3 节点高可用架构；应用 Pod 配置 Pod 反亲和性，避免多个副本调度到同一节点；配置 PodDisruptionBudget，保障节点维护时应用的最小可用副本数

6. **滚动更新策略**：配置合理的 maxSurge 和 maxUnavailable，避免更新过程中服务不可用

### 7.3 可观测性建设

1. **指标监控**：部署 Prometheus\+Grafana，监控集群节点、控制平面组件、应用 Pod 的全维度指标，设置核心指标告警

2. **日志管理**：通过 ELK/Loki 搭建统一日志平台，采集集群所有容器日志，支持检索、分析、告警，避免 Pod 删除后日志丢失

3. **链路追踪**：为微服务应用集成 Jaeger/Zipkin，实现分布式链路追踪，快速定位故障点

4. **事件监控**：采集集群异常事件，针对节点 NotReady、Pod 重启、OOM 等异常事件实时告警

### 7.4 配置与运维规范

1. 所有资源使用 YAML 文件管理，提交到 Git 版本控制，禁止直接使用 kubectl run/create 命令在生产环境创建资源

2. 使用 Helm 管理应用包，将复杂应用的资源打包为 Chart，实现版本化、可复用、一键部署 / 回滚

3. 定期升级 K8s 版本，避免使用已停止维护的版本，升级前做好备份和测试

4. 生产环境所有变更遵循灰度发布原则，先测试环境验证，再生产环境小批量灰度，最后全量发布

5. 定期备份 etcd 数据、集群配置、应用 YAML 文件，制定集群故障恢复预案并定期演练

### 7.5 YAML 配置生产避坑指南

1. 镜像必须固定版本，禁止使用`latest`标签；配置`imagePullPolicy: IfNotPresent`减少镜像拉取压力

2. 敏感信息必须用 Secret 存储，禁止明文写在 YAML 中；非敏感配置通过 ConfigMap 管理

3. ConfigMap/Secret 挂载配置文件时，必须使用`subPath`，避免覆盖容器目录内的原有文件

4. 生产建议开启 ConfigMap/Secret 的`immutable: true`，防止误改导致业务异常

5. 避免使用`nodeName`硬编码节点，优先使用`nodeSelector`或`affinity`实现调度控制

6. 为专属业务节点配置污点 \+ 容忍度，避免资源被抢占

## 核心知识点速览

- K8s 是容器编排引擎，核心解决容器化应用的集群化管理、故障自愈、服务发现、滚动更新等核心痛点

- K8s 集群分为**控制平面（Master 节点）和工作节点（Worker 节点）**，所有组件通过 kube-apiserver 通信

- 所有 K8s 资源 YAML 均遵循`apiVersion/kind/metadata/spec`四大顶层必填结构，status 为系统只读字段

- Pod 是 K8s 最小调度 / 部署单元，生产环境禁止直接创建 Pod，需通过 Deployment/StatefulSet 等控制器管理

- Deployment 是无状态应用的首选控制器，StatefulSet 适配有状态应用，提供稳定网络标识与持久化存储

- Service 为 Pod 提供固定访问地址，实现服务发现与负载均衡，Ingress 实现集群七层流量路由与 SSL 终止

- 生产环境必须为所有容器配置**资源 requests/limits**、健康检查探针，镜像禁止使用 latest 标签

- PV/PVC 实现持久化存储与应用生命周期解耦，生产推荐使用 CSI 标准存储插件

- 敏感信息必须通过 Secret 存储，非敏感配置通过 ConfigMap 管理，实现配置与镜像解耦

- 故障排查核心路径：先查集群节点状态→再查资源事件→再查 Pod 状态→再查容器日志→最后进入容器定位问题
