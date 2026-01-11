# Homelab
---
This is basic documentation of my homelab. For both personal documentation and for suggestions for improvements.

Note: my clusters are managed through this repo, please only submit a request if it is a genuine suggestion :)

## Lab

### Network Layout

```mermaid
FlowchartTD

Network_Uplink(["Network Uplink"])

Network_Uplink --> Network_Router(["Router"])
Network_Router --> Network_Switch(["5x 1Gbps Switch"])
Network_Switch --> 1-1_ProxmoxCluster001(["1-1 ProxmoxCluster001"])
Network_Switch --> 1-2_ProxmoxCluster001(["1-2 ProxmoxCluster002"])
Network_Switch --> 1-3_MiscDockerDevice(["1-3 MiscDockerDevice"])
Network_Switch --> 1-4_LAN_Management(["1-4 LAN Management Link"])
```