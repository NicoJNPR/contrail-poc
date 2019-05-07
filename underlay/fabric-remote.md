# Contrail POC
# Underlay - Contrail Fabric Management Remote

# Topology

```
     +--------------------------------------------+
     |                 br-ext                     |
     +--------------------------------------------+
        |                                      |
        |                                      |
  +-----------------+-----------------+  +-----------------+-----------------+
  |  ge-0/0/0       |                 |  |  ge-0/0/0       |                 |
  |  172.16.1.11/24 |                 |  |  172.16.1.12/24 |                 |
  |                                   |  |                                   |
  | vmx-1                             |  | vmx-2 (Gateway-2)                 |
  |  em0: 10.6.8.31                   |  |  em0: 10.6.8.32                   |
  |  lo0: 10.6.0.31                   |  |  lo0: 10.6.0.32                   |
  |  ASN: 64031                       |  |  ASN: 64032                       |
  |                                   |  |                                   |
  |  10.6.30.1/31   |                 |  |  10.6.20.2/31   |                 |
  |  ge-0/0/1       |                 |  |  ge-0/0/1       |                 |
  +-----------------+-----------------+  +-----------------+-----------------+
        |                                      |
        |                                      |
  +-----------------+-----------------+        |
  |  xe-0/0/0       |                 |        |
  |  10.6.30.0/31   |                 |        |
  |                                   |        |
  | vqfx-spine-1                      |        |
  |  em0: 10.6.8.21                   |        |
  |  lo0: 10.6.0.21                   |        |
  |  ASN: 64021                       |        |
  |                                   |        |
  |  10.6.20.0/31   |                 |        |
  |  xe-0/0/1       |                 |        |
  +-----------------+-----------------+        |
        |                                      |
        |                                      |
  +----------------+------------------+  +-----------------+-----------------+
  |  xe-0/0/0      |                  |  |  xe-0/0/0       |                 |
  |  10.6.20.1/31  |                  |  |  10.6.20.3/31   |                 |
  |                                   |  |                                   |
  | vqfx-leaf-1                       |  | vqfx-leaf-2                       |
  |  em0: 10.6.8.11                   |  |  em0: 10.6.8.12                   |
  |  lo0: 10.6.0.11                   |  |  lo0: 10.6.0.12                   |
  |  ASN: 64011                       |  |  ASN: 64012                       |
  |                                   |  |                                   |
  | xe-0/0/3  | xe-0/0/1  | xe-0/0/2  |  |  xe-0/0/1  | xe-0/0/2  | xe-0/0/3 |
  +-----------+-----------+-----------+  +------------+-----------+----------+
       |           |           |               |           |           |
       |           |           |               |           |           |
     br-r1     +--------+  +--------+      +--------+  +--------+    br-r2
       |       | bms-11 |  | bms-12 |      | bms-21 |  | bms-22 |      |
       |       +--------+  +--------+      +--------+  +--------+   compute
       |
  +----------+
  | cluster  |
  +----------+
       |
       |                         management: 10.6.8.0/24
     br-int                      loopback:   10.6.0.0/24
   10.6.8.254                    spine-leaf: 10.6.20.0/24 10.6.30.0/24
       |                         rack-1:     10.6.11.0/24
    HAProxy                      rack-2:     10.6.12.0/24

  Contrail web UI:   https://<host>:8143
  Contrail Command:  https://<host>:9091
  OpenStack Horizon: http://<host>
```


# Resource
```
                  vCPU    memory(GB)    disk(GB)    OS
vqfx-leaf-1-re      1         1           4         Junos 18.1R1.9
vqfx-leaf-1-pfe     1         2           4         Junos 18.1R1.9
vqfx-leaf-2-re      1         1           4         Junos 18.1R1.9
vqfx-leaf-2-pfe     1         2           4         Junos 18.1R1.9
vqfx-spine-1-re     1         1           4         Junos 18.1R1.9
vqfx-spine-1-pfe    1         2           4         Junos 18.1R1.9
vmx-1-vcp           1         1           4         Junos 18.3R1.9
vmx-1-vfp           4         2           4         Junos 18.3R1.9
vmx-2-vcp           1         1           4         Junos 18.3R1.9
vmx-2-vfp           4         2           4         Junos 18.3R1.9
bms-12              1         1           1         Cirros 0.4.0
bms-12              1         1           1         Cirros 0.4.0
bms-21              1         1           1         Cirros 0.4.0
bms-22              1         1           1         Cirros 0.4.0
----------------------------------------------------------------
Total              20        19          44
```
