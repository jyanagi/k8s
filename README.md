# k8s on Ubuntu 22.04 (Jammy Jellyfish)

## <span style="color: red">The Foundation: laying the groundwork</span>
Modify the hostname for your respective k8s node:
```
sudo hostnamectl set-hostname "u-k8s-cp01" // Controller Node
sudo hostnamectl set-hostname "u-k8s-w01a" // Worker Node 01a
sudo hostnamectl set-hostname "u-k8s-w01b" // Worker Node 01b
sudo hostnamectl set-hostname "u-k8s-w01c" // Worker Node 01c
exec bash
```

Create manual host records within `/etc/hosts` on each server:
```
aaa.bbb.ccc.ddd u-k8s-cp01.yourdomain.com u-k8s-cp01
aaa.bbb.ccc.xxx u-k8s-w01a.yourdomain.com u-k8s-w01a
aaa.bbb.ccc.yyy u-k8s-w01b.yourdomain.com u-k8s-w01b
aaa.bbb.ccc.zzz u-k8s-w01c.yourdomain.com u-k8s-w01c
```

The Kubernetes scheduler (kube-scheduler) is responsible for the placement of pods on k8s nodes based on resource availability. Because `kube-schedule` does not use the swap configurations, it is recommended to disable the swap space.

Disable the swap space on the OS by entering the following commands:
```
sudo swapoff -a
```
Enable and activate `overlay` and `br_netfilter` containerd modules with the following commands:
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```
Enable IP forwarding and network bridging to facilitating k8s pod networking:
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
EOF 
```
Restart `sysctl` for these changes to take effect:
```
sudo sysctl --system
```
# Enabling the Container Run Time Services (Do this on all nodes - Control & Workers)
Install the dependencies for `containerd`:
```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```
Add the Docker Repository:
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
Update aptitude (`apt`) and install `containerd.io`:
```
sudo apt update
sudo apt install -y containerd.io
```
Configure `containerd` so that it boots using the `systemd` control group (cgroup):
```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
Restart and enable `containerd`:
```
sudo systemctl restart containerd
sudo systemctl enable containerd
```
Update the `aptitude` repository for Kubernetes by adding Google's K8s repository
Note: at the time of this documentation, Xenial is the latest k8s repository (dated 19 Jan 2023 - src: https://packages.cloud.google.com/apt/dists). 
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```
Install Kubernets:
```
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
```
