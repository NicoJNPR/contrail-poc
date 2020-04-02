# Contrail POC

# 1 Overview

Pick cluster and underlay to build the POC.

### Underlay
* [Bridge](underlay/bridge.md)
  * Contrail integration deployment.
  * Contrail basic features.
* [Gateway MX](underlay/gateway-mx.md)
  * Overlay-underlay connectivity via MX.
  * Distributed fabric forwarding (gateway-less mode).
* [IP Fabric](underlay/fabric.md)
  * Contrail Fabric Management
* [IP Fabric HA](underlay/fabric-ha.md)
  * Contrail Fabric Management HA
* [Multi-Site](underlay/multi-site.md)
* [Gateway x2](underlay/gateway-x2.md)
  * Contrail gateway to corp gateway.
  * Contrail gateway to Contrail gateway (DCI).
  * Remote compute.


### Cluster
* [openstack](#a1-openstack)
* [openstack-ha](#a2-openstack-ha)
* [cfm](#a3-cfm)
* [kubernetes](#a4-kubernetes)
* [kubernetes-ha](#a5-kubernetes-ha)
* [openShift-origin](#a6-openshift)
* [multi-site](#a8-multi-site)
* [multi-site-2](#a9-multi-site-2)
* [cfm-kubernetes](#a10-cfm-and-kubernetes)
* [kubernete-mc](#a11-kubernetes-multi-cloud)


# 2 Hypervisor

Here are prerequisites for the hypervisor.

* Resource
  * The hypervisor used by this guide has 32 vCPU, 256GB memory and 2TB disk.
  * At least one NIC that has public/internet access.

* Host OS
  * CentOS 7.5 and Ubuntu 16.04.3 are validated.

* Disk partition
  * Optionally, for better disk performance, reserve a partition (1TB) for libvirt volume.
  * Partition reservation has to be done during host OS installation.
  * Here is the recommended partition schema for a 2TB disk.
    * Partition 1, 250MB, `/boot`.
    * Partition 2, 1TB, volume group `system`.
      * Logical volume `swap`, 256GB.
      * Logical volume `root`, the rest space.
    * Partition 3, the rest space (for libvirt volume).

Note: In case no partition is reserved, all VM images will be on file system.

* Kernel
  * Upgrade kernel and reboot, after installation.


# 3 Build enviroment

## 3.1 Setup hypervisor

* Install `git`.
```
yum install git
```

* Clone repository.
```
git clone https://github.com/tonyliu0592/contrail-poc.git
```

* Update the followings variables in `poc.conf`.
  * `volume_dev`
  * `ntp_server`

Note, comment out `volume_dev` if no partition is reserved.

* Setup the hypervisor.
```
cd contrail-poc
poc setup-hypervisor
```


## 3.2 VM image

Copy the following VM images `/opt/vm-image` directory.
```
CentOS-7-x86_64-GenericCloud-1805.qcow2
cirros-0.4.0-x86_64-disk.img
junos-vmx-x86-64-19.4R1.10.qcow2
vFPC-20191114.img
metadata-usb-re.img
vmx-re-hdd.qcow2
jinstall-vqfx-10-f-19.4R1.10-poc.qcow2
vqfx-19.4R1-2019010209-pfe-qemu.qcow
```

```
CentOS-7-x86_64-GenericCloud-1805.qcow2
    CentOS Linux release 7.5.1804 (Core)
    3.10.0-862.3.2.el7.x86_64

CentOS-7-x86_64-GenericCloud-1907.qcow2
    CentOS Linux release 7.6.1810 (Core)
    3.10.0-957.27.2.el7.x86_64

    3.10.0-1062.1.2.el7.x86_64
```


## 3.3 AppFormix

AppFormix and AppFormix-Flow is integrated with cluster `cfm`.

* Check Contrail documentation to find out which version of AppFormix and AppFormix Flow are required. Download the following packages from Juniper and upload them to `/opt/appformix` and `/opt/appformix-flow` directories on the hypervisor host.
```
# ls /opt/appformix/
appformix-<version>.tar.gz
appformix-dependencies-images-<version>.tar.gz
appformix-network_device-images-<version>.tar.gz
appformix-openstack-images-<version>.tar.gz
appformix-platform-images-<version>.tar.gz

# ls /opt/appformix-flow/
appformix-flows-<version>.tar.gz
appformix-flows-ansible-<version>.tar.gz
```

* Send request to `AppFormix-Key-Request.juniper.net` for AppFormix license.

* Copy the license to `/opt/appformix` directory on the hypervisor host.
```
appformix-internal-openstack-3.1.sig
```

* Update `appformix_license` in `poc.conf`.

* AppFormix and AppFormix-Flow will be deployed when build `cfm` cluster.


# 4 Deploy

## 4.1 Build underlay and cluster

* Update variables in `poc.conf` for deployment.
  * Choose underlay and cluster.
  * Set username and password for hub.juniper.net/contrail.
  * Set Contrail container tag and Contrail Command container tag.
  * Set Ansible playbook name.

* Build underlay and cluster.
```
poc build-underlay
poc build-cluster
```


## 4.2 Change underlay

* Delete existing underlay.
```
poc delete-underlay
```
#### Note: Delete underlay before updating `poc.conf`.

* Update underlay selection in `poc.conf`.

* Build new underlay.
```
poc build-underlay
```


## 4.3 Change cluster

* Delete existing cluster.
```
poc delete-cluster
```
#### Note: Delete cluster before updating `poc.conf`.

* Update cluster selection in `poc.conf`.

* Build new cluster.
```
poc build-cluster
```


# 5 Access

To access server or underlay device, login onto the host, then ssh to the hostname or management address.

HAProxy is configured to provide access to the cluster.
```
Contrail web UI:   https://<host>:8143
Contrail Command:  https://<host>:9091
OpenStack Horizon: http://<host>
AppFormix UI:      http://<host>:9000
```

Default username and password for vQFX and vMX is `root` / `Juniper`.


# 6 Notes

## 6.1 vQFX

This POC is using these vQFX RE and PFE image.
```
jinstall-vqfx-10-f-19.4R1.10-poc.qcow2
vqfx-19.4R1-2019010209-pfe-qemu.qcow
```
The PFE image is untouched. The RE image is customized with the followings.
* Root user with password `Juniper`
* SSH access
* SSH key
* DHCP on interface `em0`

With `jinstall-vqfx-10-f-18.4R1.8.qcow2`, EVPN type-2 MAC/IP route with IP shows up in `default-switch.evpn.0`. ARP request from BMS is resolved by vQFX, not being multicasted anymore. When VM sends out ARP request for BMS, vrouter doesn't resolve it, but multicasts ARP request. When vQFX resolves request and sends reply back to vrouter, it sets VNI to 0. This is invalid VNI for vrouter. So the reply is dropped by vrouter.


## 6.2 vMX

This POC is using vMX 19.4R1.10 with [trial license](https://www.juniper.net/us/en/dm/free-vmx-trial/E421992502.txt).


# Appendix A Cluster

## A.1 openstack

This is the integration with Openstack non-HA cluster.

#### Components
* Contrail Networking
* OpenStack

#### Workload
* VM

```
host          management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
contrail-1    10.6.8.1     10.6.11.1     5      64     80  CentOS 7.5-1805
openstack-1   10.6.8.2     10.6.11.2     5      48     80  CentOS 7.5-1805
compute-1     10.6.8.7     10.6.11.7     4      16     40  CentOS 7.5-1805
compute-2     10.6.8.8     10.6.11.8     4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                   18     144    240
```


## A.2 openstack-ha

This is the integration with Openstack HA cluster.

#### Components
* Contrail Networking
* OpenStack

#### Workload
* VM

```
host          management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
contrail-1    10.6.8.1     10.6.11.1     5      64     80  CentOS 7.5-1805
contrail-2    10.6.8.3     10.6.11.3     5      64     80  CentOS 7.5-1805
contrail-3    10.6.8.5     10.6.11.5     5      64     80  CentOS 7.5-1805
openstack-1   10.6.8.2     10.6.11.2     5      48     80  CentOS 7.5-1805
openstack-2   10.6.8.4     10.6.11.4     5      48     80  CentOS 7.5-1805
openstack-3   10.6.8.6     10.6.11.6     5      48     80  CentOS 7.5-1805
compute-1     10.6.8.7     10.6.11.7     4      16     40  CentOS 7.5-1805
compute-2     10.6.8.8     10.6.11.8     4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                   38     368    560
```


## A.3 cfm

This is Contrail Fabric Management with Openstack non-HA cluster.

#### Components
* Contrail Networking
* OpenStack
* CSN
* Contrail Command
* Appformix

#### Workload
* VM
* LCM BMS and non-LCM BMS

```
host              management  data        vCPU  memory   disk  OS
---------------------------------------------------------------------------
command           10.6.8.91   10.6.11.91    4      16     40  CentOS 7.5-1805
contrail-1        10.6.8.1    10.6.11.1     5      64     80  CentOS 7.5-1805
openstack-1       10.6.8.2    10.6.11.2     5      48     80  CentOS 7.5-1805
csn-1             10.6.8.3    10.6.11.3     1       8     30  CentOS 7.5-1805
compute-1         10.6.8.7    10.6.11.7     4      16     40  CentOS 7.5-1805
appformix-1       10.6.8.71   10.6.11.71    4      16     60  CentOS 7.5-1805
appformix-flow-1  10.6.8.74   10.6.11.74    4      16     60  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                      27     172    390
```


## A.4 kubernetes

This is the integration with Kubernetes non-HA cluster.

#### Components
* Contrail Networking
* Kubernetes

#### Workload
* Container

```
host          management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
cmaster-1     10.6.8.61    10.6.11.61    5      64     80  CentOS 7.5-1805
node-1        10.6.8.67    10.6.11.67    4      16     40  CentOS 7.5-1805
node-2        10.6.8.68    10.6.11.68    4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                   13      96    160
```


## A.5 kubernetes-ha

This is the integration with Kubernetes HA cluster.

#### Components
* Contrail Networking
* Kubernetes

#### Workload
* Container

```
host          management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
cmaster-1     10.6.8.61    10.6.11.61    5      64     80  CentOS 7.5-1805
cmaster-1     10.6.8.62    10.6.11.62    5      64     80  CentOS 7.5-1805
cmaster-1     10.6.8.63    10.6.11.63    5      64     80  CentOS 7.5-1805
node-1        10.6.8.67    10.6.11.67    4      16     40  CentOS 7.5-1805
node-2        10.6.8.68    10.6.11.68    4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                   23     224    320
```


## A.6 openshift

This is the integration with OpenShift Origin non-HA cluster.

#### Components
* Contrail Networking
* OpenShift Origin

#### Workload
* Container

```
host          management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
master-1      10.6.8.61    10.6.11.61    2      16     40  CentOS 7.5-1805
infra-1       10.6.8.64    10.6.11.64    5      64     80  CentOS 7.5-1805
node-1        10.6.8.67    10.6.11.67    4      16     40  CentOS 7.5-1805
node-2        10.6.8.68    10.6.11.68    4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                   15     112    200
```


## A.7 openshift-ha

This is the integration with OpenShift Origin HA cluster.

#### Components
* Contrail Networking
* OpenShift Origin

#### Workload
* Container

```
host          management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
master-1      10.6.8.61    10.6.11.61    2      16     40  CentOS 7.5-1805
master-2      10.6.8.62    10.6.11.62    2      16     40  CentOS 7.5-1805
master-3      10.6.8.63    10.6.11.63    2      16     40  CentOS 7.5-1805
infra-1       10.6.8.64    10.6.11.64    5      64     80  CentOS 7.5-1805
infra-2       10.6.8.65    10.6.11.65    5      64     80  CentOS 7.5-1805
infra-3       10.6.8.66    10.6.11.66    5      64     80  CentOS 7.5-1805
node-1        10.6.8.67    10.6.11.67    4      16     40  CentOS 7.5-1805
node-2        10.6.8.68    10.6.11.68    4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                   29     272    440
```


## A.8 multi-site

This is Contrail Fabric Management with a remote site.

#### Components
* Contrail Networking
* OpenStack
* CSN
* Contrail Command

#### Workload
* VM

```
host          management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
contrail-1    10.6.8.1     10.6.11.1     5      64     80  CentOS 7.5-1805
openstack-1   10.6.8.2     10.6.11.2     5      48     80  CentOS 7.5-1805
csn-1         10.6.8.3     10.6.11.3     1       8     30  CentOS 7.5-1805
compute-1     10.6.8.7     10.6.11.7     4      16     40  CentOS 7.5-1805
compute-r1    10.6.8.8     10.6.12.8     4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                   19     152    270
```


## A.9 multi-site-2

This is Contrail Fabric Management with a remote site.

#### Components
* Contrail Networking
* OpenStack
* CSN
* Contrail Command

#### Workload
* VM

```
host            management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
contrail-c2-1   10.6.8.51    10.6.13.51    5      64     80  CentOS 7.5-1805
openstack-c2-1  10.6.8.52    10.6.13.52    5      48     80  CentOS 7.5-1805
csn-c2-1        10.6.8.53    10.6.13.53    1       8     30  CentOS 7.5-1805
compute-c2-1    10.6.8.57    10.6.13.57    4      16     40  CentOS 7.5-1805
compute-c2-r1   10.6.8.58    10.6.14.58    4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                     19     152    270
```


## A.10 CFM and Kubernetes

This is Contrail Fabric Management and Kubernetes.

#### Components
* Contrail Networking
* OpenStack
* CSN
* Kubernetes
* Contrail Command

#### Workload
* VM
* Container
* LCM BMS and non-LCM BMS

#### Note, this is currently not supported due to a limitation from Contrail Command.

```
host          management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
contrail-1    10.6.8.1     10.6.11.1     5      64     80  CentOS 7.5-1805
openstack-1   10.6.8.2     10.6.11.2     5      48     80  CentOS 7.5-1805
csn-1         10.6.8.3     10.6.11.3     1       8     30  CentOS 7.5-1805
compute-1     10.6.8.7     10.6.11.7     4      16     40  CentOS 7.5-1805
cmaster-1     10.6.8.61    10.6.11.61    5      64     80  CentOS 7.5-1805
node-1        10.6.8.67    10.6.11.67    4      16     40  CentOS 7.5-1805
node-2        10.6.8.68    10.6.11.68    4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                   28     232    390
```


## A.11 Kubernetes Multi-Cloud

This is Kubernetes integration with multi-cloud support.

#### Components
* Contrail Networking
* Kubernetes
* CSN
* Contrail Command

#### Workload
* Container
* non-LCM BMS

```
host          management   data       vCPU  memory   disk  OS
---------------------------------------------------------------------------
cmaster-1     10.6.8.61    10.6.11.61    5      64     80  CentOS 7.5-1805
node-1        10.6.8.67    10.6.11.67    4      16     40  CentOS 7.5-1805
node-2        10.6.8.68    10.6.11.68    4      16     40  CentOS 7.5-1805
mc-gw         10.6.8.81    10.6.12.81    4      16     40  CentOS 7.5-1805
---------------------------------------------------------------------------
Total                                   17     112    200
```


# Appendix B Management address

## B.1 Underlay management address
```
            fabric-ha     fabric         multi-site     multi-site-2
-------------------------------------------------------------------------
10.6.8.11  vqfx-leaf-1    vqfx-leaf-1    vqfx-leaf-1
10.6.8.12  vqfx-leaf-2    vqfx-leaf-2    vqfx-leaf-2
10.6.8.13                                               vqfx-leaf-3
10.6.8.14                                               vqfx-leaf-4
10.6.8.21  vqfx-spine-1   vqfx-spine-1   vqfx-spine-1
10.6.8.22  vqfx-spine-2                                 vqfx-spine-2
10.6.8.31  vmx-1          vmx-1          vmx-1
10.6.8.32  vmx-2                         vmx-2
10.6.8.33                                               vmx-3
10.6.8.34                                               vmx-4
```

## B.2 Cluster management address
```
            openstack-ha  openstack    cfm          multi-site   multi-site-int
-------------------------------------------------------------------------------
10.6.8.1    contrail-1    contrail-1   contrail-1   contrail-1   contrail-1
10.6.8.2    openstack-1   openstack-1  openstack-1  openstack-1  openstack-1
10.6.8.3    contrail-2                 csn-1        csn-1        csn-1
10.6.8.4    openstack-2                             csn-r1       csn-r1
10.6.8.5    contrail-3
10.6.8.6    openstack-3                                          control-r1
10.6.8.7    compute-1     compute-1    compute-1    compute-1    compute-1
10.6.8.8    compute-2     compute-2    compute-2    compute-r1   compute-r1

            multi-site-2
-------------------------------
10.6.8.51   contrail-c2-1
10.6.8.52   openstack-c2-1
10.6.8.53   csn-c2-1
10.6.8.54   csn-c2-r1
10.6.8.57   compute-c2-1
10.6.8.58   compute-c2-r1

            kubernetes-ha  kubernetes  openshift-ha  openshift
--------------------------------------------------------------
10.6.8.61   master-1       master-1    master-1      master-1
10.6.8.62   master-2                   master-2
10.6.8.63   master-3                   master-3
10.6.8.64                              infra-1       infra-1
10.6.8.65                              infra-2
10.6.8.66                              infra-3
10.6.8.67   node-1         node-1      node-1        node-1
10.6.8.68   node-2         node-2      node-2        node-2

10.6.8.71   appformix-1
10.6.8.72   appformix-2
10.6.8.73   appformix-3
10.6.8.74   appformix-flow-1
10.6.8.81   mc-gw
10.6.8.91   command
10.6.8.95   bootstrap
10.6.8.96   aio
```


