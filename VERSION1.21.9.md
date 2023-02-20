# CREATING A KUBERNETES CLUSTER MANUALLY

## VERSION 1.21

This example uses Vagrant to create 3 local VMs. The configuration file can be found inside this repo.

### HOW TO DO IT - MASTER NODE

First, you need to disable the swap in all the nodes. To do that, you can run the following commands:

```shell
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

1 - Install `containerd` and `docker-ce`

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce containerd.io
```

2 - Enable `docker` and `containerd` services

```shell
sudo systemctl enable docker
sudo systemctl enable containerd
```

2.1 - Configure `containerd` to use `systemd` as the cgroup driver for the containerd service

```shell
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i '/^\[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options\]$/a\        SystemdCgroup = true' /etc/containerd/config.toml
```

2.2 - Configure docker group

```shell
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

3 - Install `kubeadm`, `kubelet` and `kubectl`

```shell
sudo apt update && sudo apt install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
```

It's possible to install the current version of `kubeadm`, `kubelet` and `kubectl` with:

```shell
sudo apt install -y kubelet kubeadm kubectl
```

Or install a specific version of `kubeadm`, `kubelet` and `kubectl` with:

```shell
sudo apt install -y kubelet=1.21.9-00 kubeadm=1.21.9-00 kubectl=1.21.9-00
```

You can see the available versions with:

```shell
apt list -a kubeadm
```

4 - Pull the kubernetes images

```shell
kubeadm config images pull --kubernetes-version=1.21.9
```

or without specifying the version

```shell
kubeadm config images pull
```

5 - Initialize the master node specifying the version of `kubeadm` to use

```shell
sudo kubeadm init --kubernetes-version=1.21.9 --pod-network-cidr=192.168.0.0/16
```

or use the default version of `kubeadm` installed

```shell
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

6 - Once the process is finished, run the following command:
  
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

sudo su
export KUBECONFIG=$HOME/.kube/config
exit

source ~/.bashrc
```

7 - Install a CNI plugin. In this example, we will use `Calico`:

```shell
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml

kubectl taint nodes --all node-role.kubernetes.io/control-plane-

kubectl taint nodes --all node-role.kubernetes.io/master-
```

8 - Verify that all the nodes are ready

```shell
kubectl get nodes -o wide
```

### HOW TO DO IT - WORKER NODES

Execute step 1 to 3 of the master node. Don't forget to disable the swap in all the nodes.
At step 3, don't install `kubectl`, just `kubedam` and `kubelet`.

At the end of step 3, you will get a command to join the worker nodes to the cluster. Copy and paste it in the worker nodes.

If you didn't copy the command to join to the node master, you can run this command inside the node master vm:

```shell
kubeadm token create --print-join-command
```

Execute the command in the worker nodes with sudo privileges.
