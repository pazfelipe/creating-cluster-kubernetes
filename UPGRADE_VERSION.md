# UPGRADING CLUSTER MANUALLY

## FROM VERSION 1.21.9 TO 1.22.17

### UPGRADE MASTER NODE

Execute the following commands on the master node:

```bash
sudo su
```

```bash
apt-mark unhold kubeadm
apt-get update && apt-get install -y kubeadm=1.22.17-00
apt-mark hold kubeadm
exit
```

Verify that the version of the `kubeadm` package is `1.22.17-00`:

```bash
kubeadm version
```

Verify the upgrade plan:

```bash
sudo kubeadm upgrade plan
```

Optionally, you can pull the images that will be used by the upgrade:

```bash
sudo kubeadm config images pull --kubernetes-version=v1.22.17
```

Upgrade the control plane:

```bash
sudo kubeadm upgrade apply v1.22.17
```

Upgrade kubelet and kubectl:

```bash
kubectl drain <master-node-name> --ignore-daemonsets
```

```bash
sudo su
```
  
```bash
apt-mark unhold kubelet kubectl
apt-get update && apt-get install -y kubelet=1.22.17-00 kubectl=1.22.17-00
apt-mark hold kubelet kubectl
exit
```

Restart the kubelet:

```bash
sudo systemctl deamon-reload
sudo systemctl restart kubelet
```

Bring the master node back online:

```bash
kubectl uncordon <master-node-name>
```

### UPGRADE WORKER NODES

Inside the worker node, upgrade the kubeadm:

```bash
sudo su
```

```bash
apt-mark unhold kubeadm
apt-get update && apt-get install -y kubeadm=1.22.17-00
apt-mark hold kubeadm
exit
```

Ugrade the local kubelet configuration:

```bash
sudo kubeadm upgrade node
```

Prepare the node for upgrade by marking it unschedulable and evicting the workloads. Run this command on the master node

```bash
kubectl drain <worker-node-name> --ignore-daemonsets
```

Upgrade the kubelet package:

```bash
sudo su
```

```bash
apt-mark unhold kubelet
apt-get update && apt-get install -y kubelet=1.22.17-00
apt-mark hold kubelet
exit
```

Restart the kubelet:

```bash
sudo systemctl deamon-reload
sudo systemctl restart kubelet
```

Bring the worker node back online by marking it schedulable. Run this command on the master node:

```bash
kubectl uncordon <worker-node-name>
```
