# Kubernetes

+ 192.168.231.4	控制平面节点
+ 192.168.231.2    工作节点

## 1. 准备

### 1. 安装工具

```sh
apt install apt-transport-https ca-certificates curl gnupg
```

### 2. 禁用交换分区

1. 执行

   ```sh
   swapoff -a
   ```

2. 为确保重启后仍禁用，编辑`/etc/fstab`

   ```sh
   // 注释类似
   /dev/sdX none swap sw 0 0
   ```

### 3. 加载核心模块并配置系统参数

1. 执行

   ```sh
   modprobe br_netfilter
   sysctl -w net.ipv4.ip_forward=1
   sysctl -w net.bridge.bridge-nf-call-iptables=1
   ```

2. 为持久化配置，创建配置文件

   ```sh
   echo 'net.ipv4.ip_forward=1' | tee -a /etc/sysctl.d/k8s.conf
   echo 'net.bridge.bridge-nf-call-iptables=1' | tee -a /etc/sysctl.d/k8s.conf
   sudo sysctl --system
   ```

## 2. 安装容器运行时

### 1. 安装containerd

```sh
apt install containerd
```

### 2. 配置containerd创建默认配置文件

1. 创建

   ```sh
   mkdir -p /etc/containerd
   containerd config default | tee /etc/containerd/config.toml
   ```

2. 编辑`/etc/containerd/config.toml`

   ```sh
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
       SystemdCgroup = true
   ```

3. 下载

   ```sh
   apt install  containernetworking-plugins
   ```

### 3. 重启

```sh
systemctl restart containerd
systemctl enable containerd
```

## 3. 安装kubernetes组件

### 1. 添加软件源

```sh
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list
```

### 2. 安装组件

```sh
apt-get update
apt install  kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

### 3. 启用

```sh
systemctl enable --now kubelet
```

## 4. 初始化控制平面（只在控制平面节点执行）

### 1. 初始化

```sh
kubeadm init --control-plane-endpoint 192.168.231.4:6443 --pod-network-cidr=10.244.0.0/16
```

![image-20250805093549260](https://img.lhjeong.cn/20250805093549350.png)

### 2. 配置kubectl

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 3. 验证初始化

```sh
kubectl get nodes
```

![image-20250806151951187](https://img.lhjeong.cn/20250806152025054.png)

+ 这里为NotReady为正常现象

## 5. 加入工作节点（192.168.231.2）

### 1. 获取token（控制平面节点）

```sh
kubeadm token create --print-join-command
```

![image-20250806162645024](https://img.lhjeong.cn/20250806162645056.png)

### 2. 运行join

+ 将`token`和`hash`填入

  ```sh
  sudo kubeadm join 192.168.231.4:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
  ```

### 3. 验证加入

```sh
kubectl get nodes
// 此时可以看到debian和debian2
```

## 6. 安装插件（控制平面节点执行）

+ 查看kubectl版本

  ```sh
  kubectl version
  ```

### 1. 下载yaml文件

```sh
wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

### 2. 应用flannel

1. 修改`kube-flannel.yml`

   + 示例

   ```yaml
        tolerations:
         - operator: Exists
           effect: NoSchedule
         serviceAccountName: flannel
         initContainers:
         - name: install-cni-plugin
           image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/flannel/flannel-cni-plugin:v1.4.0-flannel1
   
           volumeMounts:
           - name: cni-plugin
             mountPath: /opt/cni/bin
         - name: install-cni
           image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/flannel/flannel-cni-plugin:v1.4.0-flannel1
   
         containers:
         - name: kube-flannel
           image: docker.io/flannel/flannel:v0.24.2
   ```

   + 将`swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/flannel/flannel-cni-plugin:v1.4.0-flannel1`和`docker.io/flannel/flannel:v0.24.2`写入

2. 用docker下载然后导入到`containerd`

   1. 下载

      ```sh
      docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.33.3
      docker pull registry.aliyuncs.com/google_containers/pause:3.6
      docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/flannel/flannel-cni-plugin:v1.4.0-flannel1
      docker pull docker.io/flannel/flannel:v0.24.2
      ```

   2. 打标签

      ```sh
      docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.33.3 registry.k8s.io/kube-proxy:v1.33.3
      docker tag registry.aliyuncs.com/google_containers/pause:3.6 registry.k8s.io/pause:3.6
      ```

   3. 导出为tar

      ```
      docker save -o kube-proxy.tar registry.k8s.io/kube-proxy:v1.33.3
      docker save -o pause.tar registry.k8s.io/pause:3.6
      docker save docker.io/flannel/flannel:v0.24.2 -o flannel-v0.24.2.tar
      docker save swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/flannel/flannel-cni-plugin:v1.4.0-flannel1 -o flannel-cni-plugin-v1.4.0-flannel1.tar
      ```

   4. 导入containerd

      ```sh
      ctr -n k8s.io images import kube-proxy.tar
      ctr -n k8s.io images import pause.tar
      ctr -n k8s.io images import flannel-cni-plugin-v1.4.0-flannel1.tar
      ctr -n k8s.io images import flannel-v0.24.2.tar
      ```

   5. 查看镜像

      ```sh
      ctr -n k8s.io images ls | grep flannel
      ctr -n k8s.io images ls | grep pause
      ctr -n k8s.io images ls | grep kube-proxy
      ```

   6. 执行配置文件

      ```sh
      kubectl apply -f kube-flannel.yml
      
      // 停止（删除）
      kubectl delete -f kube-flannel.yml
      
      // 重建
      kubectl delete -f kube-flannel.yml
      kubectl apply -f kube-flannel.yml
      ```

## 7. 验证集群

1. 检查Pods

   ```sh
   kubectl get pods -A
   ```

   ![image-20250806161532440](https://img.lhjeong.cn/20250806161532521.png)

2. 部署nginx测试

   ```sh
   // 创建一个nginx应用的部署
   kubectl create deployment nginx --image=nginx
   // 暴露这个部署，创建服务
   kubectl expose deployment nginx --port=80 --type=NodePort
   // 查看当前所有服务
   kubectl get svc
   ```

   + 测试

     ```sh
     curl http://192.168.231.2:31670
     ```

     ![image-20250806163406418](https://img.lhjeong.cn/20250806163406486.png)

## 8. 配置镜像源

+ 编辑`/etc/containerd/config.toml`

  ```sh
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = ["https://registry.cn-hangzhou.aliyuncs.com"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
      endpoint = ["https://registry.cn-hangzhou.aliyuncs.com"]
  ```

+ 重启

  ```
  systemctl restart containerd
  ```

## 9. 故障排查

### 1. 检查Flannel日志

```sh
kubectl logs -n kube-flannel <flannel-pod-name>
```

### 2. 检查Kubelet日志

```sh
journalctl -u kubelet
```

### 3. 检查节点状态

```sh
kubectl describe node debian2
```



## 10. 纳入新节点

#### 1. 更新系统和安装工具

```sh
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg
```

### 2. 禁用交换分区

```sh
sudo swapoff -a
sudo nano /etc/fstab
// 注释类似/dev/sdX none swap sw 0 0
```

### 3. 配置内核模块和网络

```sh
modprobe br_netfilter
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.bridge.bridge-nf-call-iptables=1
echo 'net.ipv4.ip_forward=1' | tee -a /etc/sysctl.d/k8s.conf
echo 'net.bridge.bridge-nf-call-iptables=1' | tee -a /etc/sysctl.d/k8s.conf
sudo sysctl --system

// 这是分割用的文字
echo 'net.ipv4.ip_forward=1' | tee -a /etc/sysctl.d/k8s.conf
echo 'net.bridge.bridge-nf-call-iptables=1' | tee -a /etc/sysctl.d/k8s.conf
sysctl --system
```

![image-20250807094221889](https://img.lhjeong.cn/20250807094222001.png)

### 4. 安装容器运行时

1. 安装containerd和CNI插件

   ```sh
   apt install -y containerd containernetworking-plugins
   ```

2. 配置containerd

   ```sh
   mkdir -p /etc/containerd
   containerd config default | tee /etc/containerd/config.toml
   ```

3. 编辑`/etc/containerd/config.toml`

   ```sh
   SystemdCgroup = true
   ```

4. 配置加速器

   + 例如

     ```sh
     [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
       [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
         endpoint = ["https://registry.cn-hangzhou.aliyuncs.com"]
       [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
         endpoint = ["https://registry.cn-hangzhou.aliyuncs.com"]
     ```

5. 重启

   ```sh
   sudo systemctl restart containerd
   sudo systemctl enable containerd
   ```

### 5. 安装kubernetes组件

1. 添加软件源

   ```sh
   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key |  gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' |  tee /etc/apt/sources.list.d/kubernetes.list
   ```

2. 安装组件

   ```sh
   sudo apt update
   sudo apt install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

3. 启用

   ```sh
   systemctl enable --now kubelet
   ```

### 6. 导入镜像

```bash
#!/bin/bash
ctr -n k8s.io images import pause.tar
ctr -n k8s.io images import flannel-cni-plugin-v1.4.0-flannel1.tar
ctr -n k8s.io images import flannel-v0.24.2.tar
ctr -n k8s.io images import kube-proxy.tar
```

![image-20250807102353313](https://img.lhjeong.cn/20250807102353381.png)

### 7. 在控制平面节点获取加入命令

```sh
kubeadm token create --print-join-command
```

![image-20250807102719741](https://img.lhjeong.cn/20250807102719794.png)

+ 复制token和hash

### 8. 新节点加入集群

```sh
kubeadm join 192.168.231.4:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

![image-20250807103457978](https://img.lhjeong.cn/20250807103458087.png)

### 9. 验证新节点

1. 检查节点状态

   ```sh
   kubectl get nodes
   ```

   ![image-20250807104956222](https://img.lhjeong.cn/20250807104956299.png)

2. 检查Flannel Pods

   ```sh
   kubectl get pods -n kube-flannel
   ```

   ![image-20250807105031814](https://img.lhjeong.cn/20250807105031864.png)

### 10. Kubernetes Dashboard 部署

1. 部署Dashboard

   ```sh
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
   ```

2. 查看Dashboard Pod状态

   ```sh
   kubectl -n kubernetes-dashboard get pod -o wide
   ```

   ![image-20250811150444032](https://img.lhjeong.cn/20250811150444120.png)

3. 将`Service`修改为`NodePort`（仅供测试）

   ```sh
   kubectl -n kubernetes-dashboard edit svc kubernetes-dashboard
   ```

4. 查看分配到的端口

   ```sh
   kubectl -n kubernetes-dashboard get svc kubernetes-dashboard -o wide
   ```

5. 获取并使用Token登录Dashboard

   1. 创建并授权一个ServiceAccount

      ```sh
      kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard
      kubectl create clusterrolebinding dashboard-admin-binding \
        --clusterrole=cluster-admin \
        --serviceaccount=kubernetes-dashboard:dashboard-admin
      ```

   2. 获取Token

      ```sh
      kubectl -n kubernetes-dashboard create token dashboard-admin
      ```

      ![image-20250811144051829](https://img.lhjeong.cn/20250811144051991.png)

6. 页面展示

   ![image-20250811151106347](https://img.lhjeong.cn/20250811151106600.png)
