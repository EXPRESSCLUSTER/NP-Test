# Network Test

General HA clusters are designed to guarantee the high-availability / business-continuity for any single failure, so, what need to be tested is to make sure that the cluster can tolerate a single failure.
However, it is good to know the limits by issuing multiple consecutive failures and observing business outage.
This document describes the test for network and the expected results for HA/DR cluster.

----

## Configuration

- The cluster is made of `VM#1`, `VM#2` and `Witness`. 
- NIC `1` and NIC `1'` are used for primary heartbeat network.
- NIC `2` and NIC `2'` are used for secondary heartbeat network.
- NIC `3` and NIC `3'` are used for the ECX Forced Stop function.

![Pic.0](./image0.png)

## Terminology

- FOG : Failover group  
- ESD : Emergency shut down
- HBTO : Heartbeat time out
- VM : Virtual Machine
- PM : Physical Machine
- SW : Network switch

## Network Partition test

Start from All Green State that FOG runs on VM#1. Disconnecting VM#1 from the cluster, then VM#1 acknowledges itself as isolated and executes suicide (ESD) to maintain consistency.

![Pic.1](./image1.png)

    
| No. | Operation | Result |
|--   |--         |--      |
| 1.  | Remove the network cable from `NIC 1` and `NIC 2` | Wait for ESD of `VM#1` and failover to `VM#2`. Make sure the business continues.
| 2.  | Connect the network cable to `NIC 1` and `NIC 2` then boot `VM#1` | Wait for All Green State.
| 3.  | Remove the network cable from `NIC 1'` and `NIC 2'` | Wait for ESD of `VM#2` and failover to `VM#1`. Make sure the business continues.
| 4.  | Connect the network cable to `NIC 1'` and `NIC 2'` then boot `VM#2`| Wait for All Green State.


## Multiple Failures test

Even if an HA cluster is used, the system may stop. This test is aim to have an experience one of such cases.

**In general operation, when an HA cluster has the first failure, it is expected to be repaired before the second failure.**

This also starts from All Green State that FOG runs on VM#1. Disconnect the Witness, secondary heartbeat network, and primary heartbeat network.
In VM#1 and VM#2 view, the both consider the other to be dead. At the same time, the both can not communicate with the Witness. VM#1 considers itself as isolated from the cluster and executes suicide (ESD), and VM#2 do the same, thus the business gets outage.

![Pic.2](./image2.png)
    
| No. | Operation | Result |
|--   |--         |--      |
| 1.  | Remove the network cable from `Witness` | Wait for HBTO of `Witness` and make sure the business continues. |
| 2.  | Remove the network cable from `NIC 2` of `PM#1` (for secondary HB) | Wait for HBTO for secondary HB and make sure the business continues. |
| 3.  | Remove the network cable from `NIC 1'` of `PM#2` (for primary HB) | Wait for ESD of `SV#1` and `SV#2`. Confirm the stop of the business.|
