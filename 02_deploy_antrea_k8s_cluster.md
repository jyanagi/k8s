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

VMware Container Networking with Antrea is the commercial version of VMware's Antrea open source offering on Github. Antrea is a Kubernetes networking solution intended to be Kubernetes native. It operates at Layer3/4 to provide networking and security services for a Kubernetes cluster.

The downloaded file name is similar to: `antrea-advanced-X.Y.Z+vmware.x.zip`. 

### 1.2 Download the Antrea Interworking Adapter Files

Locate and download the `Antrea-NSX Interworking Adapter Image and Manifest Bundle`

The VMware Container Networking with Antrea, NSX Interworking connector is linking your Antrea enabled K8s cluster with NSX Manager. It is used to collect K8s inventory, statistics and logs and reports those to NSX Manager. It also translates NSX Firewall policies into Antrea Cluster Network Policies, and applies those to the K8s cluster.

The downloaded file name is similar to: `antrea-interworking-0.x.0.zip`

## 2. Extract contents of the archive file

Extract the contents of the .zip files by using the `unzip [filename.zip]` command.  

## 3. Modify the Manifests to use VMware's Image Registry

In this example, I am not using an internal registry for image distribution, so I will not be covering pushing images to a registry, such as Harbor. For information on how to do this, follow this [link](https://goharbor.io/docs/1.10/working-with-projects/working-with-images/pulling-pushing-images/).

We will need to modify 





