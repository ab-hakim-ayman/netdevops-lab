# NetDevOps Lab: 4-Node Spine-Leaf BGP Fabric with Containerlab & Ansible

An automated NetDevOps lab environment on Ubuntu using Docker, Containerlab, FRRouting (FRR), Ansible, and Jinja2.

## 🏗️ Fabric Topology

```
             +-----------------+        +-----------------+
             |     spine1      |        |     spine2      |
             |   (AS 65001)    |        |   (AS 65002)    |
             |  10.255.0.1/32  |        |  10.255.0.2/32  |
             +--------+--------+        +--------+--------+
                 eth1 |  eth2               eth1 |  eth2
                      |    \                 /   |
                      |     \               /    |
                      |      \             /     |
                 eth1 |   eth1\       eth2/      | eth2
             +--------+--------+        +--------+--------+
             |      leaf1      |        |      leaf2      |
             |   (AS 65011)    |        |   (AS 65012)    |
             |  10.255.0.11/32 |        |  10.255.0.12/32 |
             +-----------------+        +-----------------+
```

### IP Scheme & Interconnects

| Link | Interface 1 | IP 1 | Interface 2 | IP 2 | Subnet |
|------|-------------|------|-------------|------|--------|
| spine1 <-> leaf1 | `spine1:eth1` | `10.0.11.1/30` | `leaf1:eth1` | `10.0.11.2/30` | `10.0.11.0/30` |
| spine1 <-> leaf2 | `spine1:eth2` | `10.0.12.1/30` | `leaf2:eth1` | `10.0.12.2/30` | `10.0.12.0/30` |
| spine2 <-> leaf1 | `spine2:eth1` | `10.0.21.1/30` | `leaf1:eth2` | `10.0.21.2/30` | `10.0.21.0/30` |
| spine2 <-> leaf2 | `spine2:eth2` | `10.0.22.1/30` | `leaf2:eth2` | `10.0.22.2/30` | `10.0.22.0/30` |

### Management Network

- **Subnet**: `172.21.20.0/24`
- **spine1**: `172.21.20.11`
- **spine2**: `172.21.20.12`
- **leaf1**: `172.21.20.21`
- **leaf2**: `172.21.20.22`

---

## 📁 Project Structure

```text
netdevops-lab/
├── ansible.cfg                    # Ansible default configs & stdout formatting
├── configs/                       # Bootstrap FRR configs & daemon files
│   ├── daemons                    # Enables zebra and bgpd daemons
│   ├── vtysh.conf                 # Integrated vtysh config
│   ├── spine1.conf                # Initial bootstrap config
│   ├── spine2.conf                # Initial bootstrap config
│   ├── leaf1.conf                 # Initial bootstrap config
│   └── leaf2.conf                 # Initial bootstrap config
├── inventory/
│   ├── hosts.ini                  # Inventory with spines & leaves (Docker connection)
│   └── host_vars/                 # Per-node BGP & interface definitions
│       ├── spine1.yml
│       ├── spine2.yml
│       ├── leaf1.yml
│       └── leaf2.yml
├── playbooks/
│   └── deploy.yml                 # Automation playbook (Jinja2 render + vtysh apply + verification)
├── templates/
│   └── frr_bgp.j2                 # Jinja2 template for FRR interfaces & BGP peering
├── topology.clab.yml              # Containerlab 4-node topology specification
└── README.md
```

---

## 🚀 Quick Commands

### 1. Inspect Fabric
```bash
clab inspect -t topology.clab.yml
```

### 2. Run Ansible Automation Playbook
```bash
ansible-playbook playbooks/deploy.yml
```

### 3. Access FRR CLI (`vtysh`)
```bash
# Access spine1 vtysh
docker exec -it clab-netdevops-lab-spine1 vtysh

# Check BGP summary
docker exec clab-netdevops-lab-leaf1 vtysh -c "show ip bgp summary"

# Check BGP routes
docker exec clab-netdevops-lab-leaf1 vtysh -c "show ip route bgp"
```

### 4. Test Data Plane Connectivity
```bash
# Ping leaf2 loopback from leaf1
docker exec clab-netdevops-lab-leaf1 ping -c 3 -I 10.255.0.11 10.255.0.12
```

### 5. Destroy the Lab
```bash
clab destroy -t topology.clab.yml
```

---

## 🔄 CI/CD Pipeline (GitHub Actions & Self-Hosted Runner)

The lab is integrated with GitHub Actions via a Self-Hosted Runner on the local Ubuntu machine.

- **Workflow File**: `.github/workflows/pipeline.yml`
- **GitHub Repository**: [ab-hakim-ayman/netdevops-lab](https://github.com/ab-hakim-ayman/netdevops-lab)

### Pipeline Stages:
1. **Pull Request (`pre-check`)**: Runs automated Ansible syntax verification (`ansible-playbook playbooks/deploy.yml --syntax-check`).
2. **Merge / Push to `main` (`deploy-to-fabric`)**: Automatically runs the deployment playbook against the live Containerlab fabric to push configuration changes.

### Running the Local Runner:
```bash
cd ~/actions-runner
./run.sh
```
*(Or install as systemd service: `sudo ./svc.sh install && sudo ./svc.sh start`)*
