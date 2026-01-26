# Networking Model

Segments & Roles:
- Edge: HAProxy terminates TLS and routes traffic to Guacamole
- Gateway: Guacamole (web UI + guacd) brokers sessions to XRDP hosts
- Session Hosts: Linux VMs running XRDP
- Storage: NFS server accessible to session hosts

Ports (indicative):
- HAProxy: 80/443
- Guacamole (web): 8080/8443; guacd: 4822
- XRDP: 3389
- NFS: 2049
- Proxmox API: 8006

Security Boundaries:
- Edge → Gateway: restrict to required ports
- Gateway → Hosts: allow RDP (3389) from guacd only
- Hosts → Storage: allow NFS from session hosts

Note: More secure boundries to be added later production environment not live

Naming & DNS:
- `rds-edge.local`, `rds-gateway.local`, `rds-host-###.local`

Diagrams:
- See Docs/Diagrams/Remote-Desktop.mmd (Mermaid source)
