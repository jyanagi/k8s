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
>deregisterjob.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0 <br />
>interworking.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0 <br />
>interworking.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0 <br />
>interworking.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0 <br />
>interworking.yaml:          image: projects.registry.vmware.com/antreainterworking/interworking-ubuntu:0.7.0 <br />
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

You have now configured VMware's Antrea CNI with the K8s cluster; next, we need to integrate our Antrea CNI with NSX.

## 5 Integrate VMware Antrea CNI with NSX

To connect and integrate NSX with our upstream k8s cluster, we need to create a certificate that is associated with an NSX principle identity account. In production deployments, these certificates are issued by a trusted Root CA (i.e., DigiCert, Entrust, IdenTrust); however, in this example, we will be using self-signed certificates for simplicity and demonstration purposes.

### 5.1 Create Self-Signed Certificate

Issue the following commands to generate a private key `(.key)` and an x.509 certificate `(.crt)`:
```
#Create and change the working directory to 'certs' 
mkdir certs && cd certs

#Generate the RSA Private Key (2048 is default modulus size)
openssl genrsa -out u-k8s-cluster.key

#Generate the Certificiate Signing Request (CSR)
openssl req -new -key u-k8s-cluster.key -out u-k8s-cluster.csr -subj "/C=US/ST=VA/L=Richmond/O=ProjectONEStone/OU=NSX/CN=Ubuntu Antrea K8s Cluster"

#Issue the self-signed certificate using the CSR and signing with the private key
openssl x509 -req -days 365 -sha256 -in u-k8s-cluster.csr -signkey u-k8s-cluster.key -out u-k8s-cluster.crt
```
You will now have three files in the `./certs` directory:  `u-k8s-cluster.key`, `u-k8s-cluster.csr`, and `u-k8s-cluster.crt`

## 5.2 Modify the Antrea Interworking Bootstrap Configuration

Now that we have our self-signed certificate, we must modify the bootstrap configuration file for Antrea.

Navigate to the `~/antrea-interworking-0.7.0` directory (replace with your absolute path) and you should see the `bootstrap-config.yaml`

Do not forget to replace the IP address of the `nsxManager` variable with your own NSX manager IP address.

```
clusterName="u-k8s-cluster"
nsxManager="10.15.11.20" # Replace with your NSX Cluster IP address

sed -i "s|dummyClusterName|${clusterName}|g" bootstrap-config.yaml
sed -i "s|dummyNSXIP1, dummyNSXIP2, dummyNSXIP3|${nsxManager}|g" bootstrap-config.yaml
sed -i "35 s|$| $(cat ~/certs/u-k8s-cluster.crt | base64 -w 0)|g" bootstrap-config.yaml
sed -i "37 s|$| $(cat ~/certs/u-k8s-cluster.key | base64 -w 0)|g" bootstrap-config.yaml
```
What these commands did was modify the `bootstrap-config.yaml` file and add the Ubuntu k8s cluster certificate and private key information so that we can integrate our kubernetes cluster with NSX.

## 5.3 Apply the Bootstrap and Interworking configurations

Ensure you are in the `~/antrea-interworking-0.7.0` directory
```
cd ~/antrea-interworking-0.7.0
kubectl apply -f bootstrap-config.yaml -f interworking.yaml
```
If successful, it will create the `vmware-system-antrea` namespace within the cluster, create the NSX secrets, configure the cluster role bindings, and map the configurations for NSX integration with the Antrea CNI.

You can verify by issuing the command:
```
kubectl get pods -n vmware-system-antrea
```
You should see similar output...

| NAME                          | READY | STATUS  | RESTARTS | AGE |
| :-----------------------------|:------|:--------|:---------|-----|
| interworking-6c8c5b78b4-td9l8 | 4/4   | Running | 0        | 7m  |



## 5.4 Create Principal Identity User Account in NSX

Copy the contents of the `u-k8s-cluster.crt` file; we will need to paste this into our Principal Identity User configuration within NSX.

Open a web browser to to your NSX Manager and navigate to `System Tab > Settings: User Management > User Role Assignment` and click `ADD PRINCIPAL IDENTITY`.

Configure the following
| User/User Group Name: | `u-k8s-cluster`           |
|:----------------------|:-------------------       |
| Roles:                | `Enterprise Admin`        |
| Node Id:              | `u-k8s-cluster`           |
| Certificate PEM       | Paste Certificate Content |

<p align=center>
  <img src=https://bit.ly/nsx-principal-identity>
</p>

You should now be able to go into NSX's inventory and see your kubernetes cluster objects (pods and services)!

As you deploy new namespaces and pods, these will now appear in the inventory and you will be able to apply security policies to the container workloads.










