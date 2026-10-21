# NetDevOps Lab - Multi-Project Network Automation Monorepo

Welcome to the **NetDevOps Lab** repository. This repository is organized as a clean, modular multi-project monorepo containing two distinct, enterprise-grade network automation projects:

```text
netdevops-lab/
├── spine-leaf-fabric/            # Project 1: Data Center EVPN / eBGP Spine-Leaf Fabric
│   ├── topology.clab.yml         # Containerlab 2-Spine / 2-Leaf FRRouting topology
│   ├── docker-compose.netbox.yml # Local NetBox SSOT + PostgreSQL + Redis
│   ├── configs/                  # Prometheus, Alertmanager, Grafana provisioning
│   ├── inventory/                # Dynamic NetBox inventory & static host_vars
│   ├── playbooks/                # Ansible deployment & pre-check playbooks
│   ├── templates/                # Jinja2 templates for FRR frr.conf
│   └── scripts/                  # Pydantic schema validation & NetBox seeding
│
├── corporate-office/             # Project 2: Multi-Branch Corporate Network Automation
│   ├── inventory/                # Host vars (hq_router, branch2_router) & NetBox dynamic inv
│   ├── templates/                # Master MikroTik RouterOS v7 Jinja2 template
│   ├── scripts/                  # Pydantic schema validation, config generator, NetBox sync
│   ├── generated_configs/        # Pre-rendered production .rsc scripts (HQ & Branch 2)
│   ├── playbooks/                # Automated deployment & end-to-end ping validation
│   └── README.md                 # Complete MikroTik architecture & runbook
│
└── .github/                      # Centralized CI/CD GitHub Actions pipelines
    └── workflows/
        └── pipeline.yml          # Dual-project linting, schema validation & automated deploy
```

---

## 🏛️ Project Summaries

### 1. [spine-leaf-fabric](./spine-leaf-fabric/) — Data Center Fabric Automation
* **Architecture:** 2-Spine / 2-Leaf CLOS topology built on FRRouting (FRR 9.1+) and Containerlab.
* **Underlay / Routing:** eBGP underlay with private ASNs (Spine: 65001-65002, Leaf: 65011-65012) and BFD fast convergence.
* **Source of Truth:** NetBox (DCIM & IPAM) providing dynamic inventory and config context.
* **Observability:** Prometheus metrics scraping via `frr-exporter`, Grafana dashboards, and Alertmanager.
* **Safety:** Automated pre-change configuration backup, atomic config reload (`frr-reload.py`), and automated rollback on validation failure.
* 📖 **Quickstart Guide:** See [spine-leaf-fabric/README.md](./spine-leaf-fabric/README.md).

```bash
# Navigate to fabric directory
cd spine-leaf-fabric

# Deploy lab topology
sudo clab deploy -t topology.clab.yml

# Run Ansible configuration & validation
ansible-playbook playbooks/deploy.yml
```

---

### 2. [corporate-office](./corporate-office/) — Multi-Branch Corporate Network Automation
* **Architecture:** Enterprise HQ (Dhaka) & Branch (Chittagong) branch office network powered by MikroTik RouterOS v7.
* **Dual ISP WAN:** Active/Backup WAN with recursive route failover (Targeting `8.8.8.8` via ISP1 and `1.1.1.1` via ISP2).
* **Site-to-Site VPN:** Fully encrypted WireGuard tunnel (`10.254.0.0/30`) with MTU 1420 and static routing between branches.
* **Segmentation:** 9 distinct Departmental VLANs with standard 802.1Q hardware bridge filtering:
  * HR (VLAN 10), Admin (VLAN 20), Management (VLAN 30), Employee (VLAN 40), Staff (VLAN 50), CCTV (VLAN 60), VoIP (VLAN 70), Printer (VLAN 80), Guest WiFi / Hotspot (VLAN 90).
* **Security & Access Control:** Strict stateful firewall, inter-VLAN isolation (preventing guest/hotspot and employee access to CCTV/Management), and NAT overload (masquerade) with dynamic WAN out-interface lists.
* **Source of Truth & Tooling:** NetBox dynamic inventory, Jinja2 template rendering, Pydantic v2 schema validation, and Ansible deployment.
* 📖 **Quickstart Guide:** See [corporate-office/README.md](./corporate-office/README.md).

```bash
# Navigate to corporate office directory
cd corporate-office

# Validate schema with Pydantic
python3 scripts/validate_corp_schema.py

# Generate RouterOS .rsc configuration scripts
python3 scripts/generate_configs.py

# Sync sites, routers, and 18 VLAN prefixes to NetBox
python3 scripts/netbox_corp_sync.py

# Deploy to live MikroTik devices via Ansible
ansible-playbook playbooks/deploy_corp.yml
```

---

## 🚀 Shared Services & NetBox IPAM

Both projects are integrated with a shared local NetBox instance running at `http://localhost:8000`:
- **Spine-Leaf Fabric:** Filtered under site `lab-dc1` and roles `spine`, `leaf`.
- **Corporate Office:** Filtered under sites `hq-dhaka`, `branch-chittagong` and role `gateway-router`.

NetBox API Token: `0123456789abcdef0123456789abcdef01234567`.

To start or inspect the shared NetBox stack:
```bash
cd spine-leaf-fabric
docker compose -f docker-compose.netbox.yml up -d
```
