---
title: kubernetes安装过程
date: 2021-05-04 19:42:53
categories:
tags:
---
# 背景
由于本人对linux系统很感兴趣，对运维方面的技术也很感兴趣。如今软件部署进入了容器化时代，技术就要跟上时代的步伐。
软件的部署方案大概分为三种阶段：
- 第一阶段，是在目标机上安装好软件部署所需要的环境，比如说java环境，nginx环境等等，然后再将新版本的软件服务启动起来；
   这个方法导致了一些问题，那便是如果有多台物理主机进行分布式部署，将需要在每台主机上进行同样的操作，虽然可以通过写脚本来尽量避免这种工作，但是依然十分复杂，对于要新增一种软件部署也要进行大量重复工作；
- 第二阶段，是使用容器技术，比如docker，将新版本的软件打包，然后在目标机器上拉取最新的软件包，并启动；
   这个方法依旧还存在问题，那就是会需要在每台物理主机上都进行拉取软件包并启动的操作，依旧有重复性的工作；
- 最后，也就是目前比较流行的一种方式，使用[kubernetes(简称k8s)](https://kubernetes.io/zh/)集群，这种方法比较简单，在第二阶段的基础上，使用了k8s管理工具，只需要在master节点主机上部署新版本软件包，该工具将会自动在所有子节点上创建新的service，并不需要重复性的工作，这一切都被k8s自动完成了；
   当然k8s并不只有这个功能，还有一些其他非常强大的功能，就不一一细说了。
<!-- more -->
# 准备工作
## 节点机器
安装k8s集群工作至少需要三台独立的机器才有意义，一台或者更多master节点，两台或者更多node节点。
有两个方案：
- 如果手头有足够的旧电脑，那么可以使用旧电脑安装k8s集群工具；
- 如果手头没有多台电脑，那么考虑第二种方案，在主机上安装三台虚拟机模拟k8s集群。
本人使用的是第二种方案，由于多台虚拟机比较耗费资费，于是我将自己的台式机硬件升级了一下，内存32G，固态750G。
## 安装镜像包(可以在安装完kubeadm后进行)
由于k8s集群工具安装过程中需要连接网络下载一些必需的镜像，而这些镜像所在的网站在国内被墙了，所以需要事先准备离线镜像包，主要包括kubeadm init所需要的镜像包，以及k8s网络插件所要下载的镜像包。我使用的是[flannel](https://github.com/flannel-io/flannel)网络插件，这个插件需要下载 quay.io/coreos/flannel 镜像包；可以事先准备好。本人是在另一台linux上使用vpn连接外网下载的所有镜像包，然后通过`docker save [tags|ids] > k8s-image.tar`导出，拷贝到虚拟机里，使用`docker load < k8s-image.tar`导入虚拟机里。
## 虚拟机安装及配置
本人安装了三台debian 10虚拟机，关闭了交换分区，并配置了网络地址转换和仅主机两种网络，其中仅主机网络是局域网，并固定设置了三台虚拟机的ip地址。
# 安装进行
## 安装docker，以及基本k8s工具kubeadm, kubectl, kubelet
本步骤网上很多教程，在这里我不再赘述，需要注意的是，这个步骤需要在所有虚拟机上都进行，而且要让docker和kubectl自动启动。这一步骤可以将准备的镜像包导入三台虚拟机，需要下载的镜像包可以使用命令`kubeadm config images list`查看，我的机器上输出的包有：
k8s.gcr.io/kube-apiserver:v1.21.0
k8s.gcr.io/kube-controller-manager:v1.21.0
k8s.gcr.io/kube-scheduler:v1.21.0
k8s.gcr.io/kube-proxy:v1.21.0
k8s.gcr.io/pause:3.4.1
k8s.gcr.io/etcd:3.4.13-0
k8s.gcr.io/coredns/coredns:v1.8.0
然后在可连接外网的机器上进入[docker hub](https://hub.docker.com/)上搜索具体的包，使用`docker pull IMAGE[:TAG]`，下载**对应的版本**，然后使用如`docker tag kube-apiserver:v1.21.0 k8s.gcr.io/kube-apiserver:v1.21.0`的命令将对应的包打上k8s所需要的包相应的tag，使用`docker rmi [imageID]`删除旧的包，flannel包需要在安装网络插件时才知道需要安装的具体版本(我的机器上是 quay.io/coreos/flannel:v0.14.0-rc)。然后根据步骤**安装镜像包**中所描述的命令导出镜像，并将镜像加载进入三台虚拟机的docker内。
## master节点工作
- 在master节点进行k8s的初始化，使用命令`kubeadm init --kubernetes-version=v1.21.0 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.56.5`，其中kubernetes-version参数改为你本地安装的k8s版本，--pod-network-cidr参数取决于你要使用的网络插件，这里我使用的是flannel，所以地址为这个，--apiserver-advertise-address参数指向master节点的地址；
- 这一步骤如果出错了，解决了错误之后，需要执行`kubeadm reset`重置，然后再执行init命令；
- 执行init成功后，会提示进行一些操作，`mkdir -p $HOME/.kube`，`sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`，`sudo chwon $(id -u):$(id -g) $HOME/.kube/config`，三条命令；执行完成后可以通过`kubectl get nodes`查看当前的所有节点，目前只有master节点；通过执行`kubectl get pods --all-namespaces`查看所有的pods，可以看到coredns还处于pending状态，因为没有安装网络插件；
- 执行`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`安装网络插件，这里也可以将配置文件下载至本地，然后执行；在这里会生成一个pod，kube-flannel-ds-amd64-***啥的，可以通过`kubectl get pods --all-namespaces`查看，这里如果无法访问外网会卡住，因为会下载一个镜像包，通过`kubectl describe pods kube-flannel-ds-amd64-[ID] -n kube-system`查看具体情况和镜像包名为 quay.io/coreos/flannel:v0.14.0-rc,版本号因人而异，在能连接外网的电脑上下载该镜像包并导出，然后导入到三台虚拟机的docker里；
- 这个时候apply命令就会自动执行成功了，不用再次执行。
master节点初始化完成。
## 添加node节点
- 在master节点执行`kubeadm token create --print-join-command`生成加入节点命令；
- 在node节点执行 1 生成的命令就可以加入相应集群；
- 在master节点执行`watch kubectl get pods --all-namespaces`动态查看pods，可以看到多了kube-flannel-ds-amd64-\*\*\*和kube-proxy-\*\*\*的pod；
- 当上述状态为Running的时候就执行成功了，执行`kubectl get nodes`查看所有节点。
# 总结
在国内安装k8s确实会很麻烦，很多镜像包需要外网才能下载，而且虚拟机目前我还没找到共享主机vpn的方法，镜像包还需要离线下载导出并且修改镜像包的tag为正确的tag，再到虚拟机导入docker里，比较麻烦。不过有一个好处，就是公司的服务器基本都只支持内网访问，所以只能离线方式安装k8s，也算是为下一步应用到实体机做预备了。
另外，安装k8s集群如果是采用多台虚拟机的方案，会对主机要求比较高，可以自行升级硬件设备。
