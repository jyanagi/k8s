# k8s on Ubuntu 22.04 (Jammy Jellyfish)

##The Foundation: laying the groundwork
Modify the hostname for your respective k8s node:
```
sudo hostnamectl set-hostname "u-k8s-cp01"
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
Restart `sysctl` for these changes to take effect (ignore any errors you see on `sysctl: setting key "net.ipv4.conf.all.: Invalid Argument`:
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
## Initialize the k8s cluster [ON CONTROL NODE ONLY]

Run the kubeadm command from the **control node** only:
```
sudo kubeadm init --control-plane-endpoint=uk8s-cp01.yourdomain.com
```
If successful, you should receive a message similar to the following:

>Your Kubernetes control-plane has initialized successfully!
>
>To start using your cluster, you need to run the following as a regular user:
>
>  **mkdir -p $HOME/.kube
>  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
>  sudo chown $(id -u):$(id -g) $HOME/.kube/config**
>
>Alternatively, if you are the root user, you can run:
>
>  export KUBECONFIG=/etc/kubernetes/admin.conf
>
>You should now deploy a pod network to the cluster.
>Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
>  https://kubernetes.io/docs/concepts/cluster-administration/addons/
>
>You can now join any number of control-plane nodes by copying certificate authorities
>and service account keys on each node and then running the following as root:
>
>  kubeadm join uk8s-cp01.yourdomain.com:6443 --token xo5dno.ptesnjr1hdm1skcc \
>        --discovery-token-ca-cert-hash sha256:3deb7a0571ddeb5728ca9378795fb7809fb1fe03ddc2d0fc68afc399a26810e7 \
>        --control-plane 
>
>Then you can join any number of worker nodes by running the following on each as root:
>
>kubeadm join uk8s-cp01.yourdomain.com:6443 --token xo5dno.ptesnjr1hdm1skcc \
>        --discovery-token-ca-cert-hash sha256:3deb7a0571ddeb5728ca9378795fb7809fb1fe03ddc2d0fc68afc399a26810e7

Note the `kubeadm joinuk8s-cp01.yourdomain.com:6443 --token xo5dno.ptesnjr1hdm1skcc \ --discovery-token-ca-cert-hash sha256:3deb7a0571ddeb5728ca9378795fb7809fb1fe03ddc2d0fc68afc399a26810e7` command from the output above.

This will be applied to every worker node to join them to the k8s cluster. 

But first, we must issue the following commands to interact with the k8s cluster:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
To view the nodes within the k8s cluster, issue the command:
```
kubectl get nodes
```
You should see an output of the following (or similar)

>NAME             STATUS     ROLES           AGE   VERSION <br /> 
>napp-uk8s-cp01   NotReady   control-plane   1m   v1.26.1

Notice how the status for your Controller Node is `NotReady`. That's because there are no workers joined to the cluster.

## Join the Worker Nodes to the k8s cluster [ON WORKER NODES ONLY]
Copy the `kubeadm join` output of the kubernetes initialization process and execute via the command:
```
kubeadm join uk8s-cp01.yourdomain.com:6443 --token xo5dno.ptesnjr1hdm1skcc \ --discovery-token-ca-cert-hash sha256:3deb7a0571ddeb5728ca9378795fb7809fb1fe03ddc2d0fc68afc399a26810e7
```
