# NOTE: To be performed on the CONTROL NODE only
# Overview

Antrea Container Network Interface (CNI) [antrea.io] is a k8s-native project that provides programmable networking and security policies for pod workloads. Antrea enables k8s pod networking with IP overlay, software-defined networks via GENEVE or VXLAN encapsulation and is built on the Open vSwitch (OVS). 

The release of VMware NSX-T 3.2 offers customers to integrate their Antrea clusters with NSX-T for centralized container policy management and visibility extending their ability to manage, tag, group and secure their Antrea clusters through an adapter - the Antrea NSX Adapter.

## Architecture

<p align="center">
<img src=https://bit.ly/nsx-antrea>
</p>

## 1. Download the Antrea NSX Image and Manifests
Because we will be integrating NSX with Antrea, we cannot use the open source Antrea CNI image. Instead, we will use the images provided by VMware. 

This requires you to have NSX entitlements to download the VMware Antrea Enterprise software via the customer connect portal: <a href="https://customerconnect.vmware.com/downloads/info/slug/networking_security/vmware_antrea/1_x">Vmware Antrea Enterprise 1.x</a>

**Note: *this link will not work if you do not have entitlements.***

You will have several options within the `Product Downloads` tab based on your Linux distribution type.

### 1.1 Download the Antrea Deployment Files

Because we have deployed the cluster on Ubuntu 22.04 (Jammy Jellyfish), we will download and use the `VMware Container Networking with Antrea (Advanced) ? Debian Image and Deployment` option (*UBI is for RedHat/CentOS*).  

VMware Container Networking with Antrea is the commercial version of VMware's Antrea open source offering . Antrea is a Kubernetes networking solution intended to be Kubernetes native. It operates at Layer3/4 to provide networking and security services for a Kubernetes cluster.

The downloaded file name is similar to: `antrea-advanced-1.7.1+vmware.x.zip`. Note that this version will change as this documentation ages.

### 1.2 Download the Antrea Interworking Adapter Files

Locate and download the `Antrea-NSX Interworking Adapter Image and Manifest Bundle`

The VMware Container Networking with Antrea, NSX Interworking connector is linking your Antrea enabled K8s cluster with NSX Manager. It is used to collect K8s inventory, statistics and logs and reports those to NSX Manager. It also translates NSX Firewall policies into Antrea Cluster Network Policies, and applies those to the K8s cluster.

The downloaded file name is similar to: `antrea-interworking-0.7.0.zip`. Note that this version will change as this documentation ages.

## 2. Extract contents of the archive file

Extract the contents of the .zip files by using the `unzip [filename.zip]` command.  

If you do not have unzip, you can install it using the following command:
```
sudo apt install -y unzip
```
## 3. Modify the NSX Antrea Manifests to use VMware's Image Registry

In this example, I am not using an internal registry for image distribution, so I will not be covering pushing images to a registry, such as Harbor. For information on how to do this, follow this [link](https://goharbor.io/docs/1.10/working-with-projects/working-with-images/pulling-pushing-images/).

The URL for the Container Images on VMware's distribution Harbor can be found on the [*VMware Container Networking with Antrea Release Notes*](https://docs.vmware.com/en/VMware-Container-Networking-with-Antrea/1.5.0/rn/vmware-container-networking-with-antrea-150-release-notes/index.html)

We will be using the following URLs:

Antrea images:
> projects.registry.vmware.com/antreainterworking/antrea-advanced-debian:v1.7.1_vmware.1

Antrea-NSX images:
> projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0

### 3.1 Modify Antrea Deployment Manifest

Navigate to the directory `~/antrea-advanced-1.7.1+vmware.1/manifests` (Your location/absolute path may be different)

```
cd ~/antrea-advanced-1.7.1+vmware.1/manifests
```
We need to replace the container image library where the `antrea-advanced-v1.7.1+vmware.1.yml` manifest is indexing its container images from; i.e., `image: "antrea/antrea-advanced-debian:v1.7.1_vmware.1"`.

Instead, we want to point it to VMware's distribution Harbor registry. Use the following command to replace all image indexes within this document:

```
sed -i 's/image: \"antrea\//image: \"projects.registry.vmware.com\/antreainterworking\//g' antrea-advanced-v1.7.1+vmware.1.yml
```
Confirm by issuing the command
```
grep "images: " antrea-advanced-v1.7.1+vmware.1.yml
```

You should see similar output:

>image:  \"projects.registry.vmware.com/antreainterworking/antrea-advanced-debian:v1.7.1_vmware.1\"<br />
>image:  \"projects.registry.vmware.com/antreainterworking/antrea-advanced-debian:v1.7.1_vmware.1\"<br />
>image:  \"projects.registry.vmware.com/antreainterworking/antrea-advanced-debian:v1.7.1_vmware.1\"<br />
>image:  \"projects.registry.vmware.com/antreainterworking/antrea-advanced-debian:v1.7.1_vmware.1\"<br />
>image:  \"projects.registry.vmware.com/antreainterworking/antrea-advanced-debian:v1.7.1_vmware.1\"<br />

### 3.2 Modify Antrea Interworking Manifest

Navigate to the directory `~/antrea-interworking-0.7.0` (Your location/absolute path may be different)
```
cd ~/antrea-interworking-0.7.0
```

We need to replace the container image library for both `interworking.yaml` and `deregisterjob.yaml` files are indexing their images from; i.e., `image: vmware.io/antrea/interworking:0.7.0`.

Like the previous step, we want to point to VMware's distribution Harbor registry. Use the following command to replace all image indexes within these documents:

```
sed -i 's/image: vmware.io\/antrea\/interworking:0.7.0/image: projects.registry.vmware.com\/antreainterworking\/interworking-ubuntu:0.7.0/g' interworking.yaml 
sed -i 's/image: vmware.io\/antrea\/interworking:0.7.0/image: projects.registry.vmware.com\/antreainterworking\/interworking-ubuntu:0.7.0/g' deregisterjob.yaml
```
Confirm by issuing the command:
```
grep "images: " interworking.yaml deregisterjob.yaml
```
You should see similar output:
>deregisterjob.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0
>interworking.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0
>interworking.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0
>interworking.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0
>interworking.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0
>interworking.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0

## 4. Deploy Antrea as the CNI for the Kubernetes Cluster

Navigate to the same directory as the `antrea-advanced-v1.7.1+vmware.1.yml` file and run the following `kubectl` command:
```
kubectl apply -f antrea-advanced-v1.7.1+
```
To verify that Antrea was successfully deployed to the k8s cluster, use the command `kubectl get pods -n kube-system | grep antrea`.  You can even append `-w` to *watch* the status change.  There should be an instance of the Antrea controller along with an instance of Antrea agents (one for each node) with a status of "Running".  See below for example output: 

| NAME                                   | READY  | STATUS   | RESTARTS  | AGE  |
|:---------------------------------------|:-------|:---------|:----------|:-----|
| antrea-agent-68rgw                     | 2/2    | Running  | 0         | 33s  |
| antrea-agent-8wcgs                     | 2/2    | Running  | 0         | 33s  |
| antrea-controller-6bf74dd6f8-c9twh     | 1/1    | Running  | 0         | 33s  |

Congratulations!  You have now configured VMware's Antrea CNI with the K8s cluster; next, we need to integrate our Antrea CNI with NSX.





