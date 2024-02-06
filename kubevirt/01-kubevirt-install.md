# Kubevirt On Minikube 笔记

## 安装 minikube

执行如下命令来安装最新版本的 minikube：

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```

## 准备 minikube Kubernetes 环境

执行如下命令来启动一个 minikube Kubernetes 环境：

```shell
minikube start --cni=flannel --force
```

## 部署 KubeVirt

安装 KubeVirt operator：

```shell
export VERSION=$(curl -s https://storage.googleapis.com/kubevirt-prow/release/kubevirt/kubevirt/stable.txt)
echo $VERSION
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
```

安装 KubeVirt CRD：

```shell
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
```

验证组件部署成功：

```shell
kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.phase}"
kubectl get all -n kubevirt
```

## 安装 Virctl

执行如下命令来安装 Virtcl：

```shell
VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
echo ${ARCH}
curl -L -o virtctl https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-${ARCH}
chmod +x virtctl
sudo install virtctl /usr/local/bin
```

## 使用 KubeVirt

首先创建一个 VM：

```shell
wget https://kubevirt.io/labs/manifests/vm.yaml
less vm.yaml
kubectl apply -f https://kubevirt.io/labs/manifests/vm.yaml
```

查看并启动 VM：

```shell
kubectl get vms
kubectl get vms -o yaml testvm

virtctl start testvm

kubectl get vmis
kubectl get vmis -o yaml testvm
```

登录 VM 终端：

```shell
virtctl console testvm
```

关机 VM：

```shell
virtctl stop testvm
```

删除 VM：

```shell
kubectl delete vm testvm
```

