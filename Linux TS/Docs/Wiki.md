## Overview



### Network Layout

# Overview

This wiki page links into the docs-first structure for the Remote Desktop Platform on Proxmox.

- Architecture overview: see [Docs/Architecture/Platform.md](Docs/Architecture/Platform.md)
- Networking model: see [Docs/Architecture/Networking.md](Docs/Architecture/Networking.md)
- Runbooks: see [Docs/Runbooks](Docs/Runbooks)

Network Layout (Homelab):

```mermaid
flowchart LR
	Router1((Router))

	subgraph Switches
		Switch1((Switch))
		Switch2((Switch))
	end

	subgraph Proxmox Cluster 001
		ProxmoxCluster001((Proxmox Cluster 001))
	end

	subgraph Proxmox Cluster 002
		ProxmoxCluster002((Proxmox Cluster 002))
	end

	Router1 --> Switch1
	Router1 --> Switch2
	Switch1 --> ProxmoxCluster001
	Switch2 --> ProxmoxCluster002
```
