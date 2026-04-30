# OwnCloud Deployment on Azure — Secure Two-Tier Architecture

![Azure](https://img.shields.io/badge/Cloud-Microsoft%20Azure-0078D4?logo=microsoftazure&logoColor=white)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu-E95420?logo=ubuntu&logoColor=white)
![MySQL](https://img.shields.io/badge/Database-MySQL-4479A1?logo=mysql&logoColor=white)
![OwnCloud](https://img.shields.io/badge/App-OwnCloud-0082C9?logo=owncloud&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

A hands-on cloud infrastructure project that provisions a secure, production-style file hosting solution on Microsoft Azure. The architecture separates application and database tiers across public and private subnets, applying real-world security principles such as least-privilege network access, bastion host access patterns, and NSG-enforced traffic control.

---

## 📐 Architecture Overview

![Architecture Diagram](architecture.png)

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Cloud Provider | Microsoft Azure |
| Virtual Network | Azure VNet, Public & Private Subnets |
| Firewall / Access Control | Network Security Groups (NSG) |
| Outbound Connectivity | Azure NAT Gateway |
| Admin Access | Bastion Host (SSH/RDP) |
| Application Server | OwnCloud on Ubuntu 22.04 |
| Database Server | MySQL on Ubuntu 22.04 |
| OS | Ubuntu Linux 22.04 LTS |

---

## ✅ Implementation Steps

### 1. Virtual Network & Subnet Design
Created an Azure Virtual Network with a `/16` address space, divided into two subnets:
- **Public Subnet** (`10.0.1.0/24`) — hosts the internet-facing OwnCloud application server
- **Private Subnet** (`10.0.2.0/24`) — hosts the MySQL database server with no public IP, completely isolated from the internet

### 2. MySQL Setup on Private Subnet
- Provisioned an Ubuntu 22.04 VM with **no public IP** in the Private Subnet
- Connected via the **App Server acting as a Bastion Host** — SSH into the App Server first, then jump across to the DB Server using its private IP
- Configured a **NAT Gateway** on the Private Subnet to allow the VM to pull packages (`apt install`) without being publicly reachable inbound
- Installed, secured, and configured MySQL — created a dedicated database and user for OwnCloud

### 3. OwnCloud Setup on Public Subnet
- Provisioned an Ubuntu 22.04 VM with a **Public IP** in the Public Subnet
- Installed OwnCloud and all required dependencies (Apache, PHP, etc.)
- Configured OwnCloud to connect to the MySQL instance on the Private Subnet via its private IP

### 4. Network Security Group (NSG) Configuration
Applied least-privilege inbound/outbound rules:

| Rule | Port | Direction | Source | Purpose |
|---|---|---|---|---|
| Allow HTTPS | `443` | Inbound | Internet | User access to OwnCloud |
| Allow MySQL | `3306` | Inbound | Public Subnet | App server → DB communication |
| Allow SSH | `22` | Inbound | Admin IP | Bastion Host management |
| Allow RDP | `3389` | Inbound | Admin IP | Bastion Host management |
| Deny All | `*` | Inbound | Any | Default deny everything else |

### 5. Verification & Testing
- Accessed the OwnCloud login page via the app server's public IP in a browser over HTTPS
- Logged in and confirmed full application functionality
- Verified the database connection was active between OwnCloud and the MySQL server on the private subnet

---

## 🔒 Security Design Decisions

**Why a Private Subnet for MySQL?**
Databases should never be directly reachable from the internet. Placing MySQL in a private subnet means even if the app server is compromised, the attacker cannot reach the database without traversing the NSG.

**Why does the App Server double as a Bastion Host?**
Rather than opening SSH directly to the database server from the internet, the App Server acts as a jump box — admins SSH into the App Server first, then hop into the DB Server using its private IP. This means the DB Server is never directly reachable from outside the VNet, reducing the attack surface significantly.

**Why a NAT Gateway?**
The private subnet VMs need outbound internet access to download packages and receive OS updates, but should never accept unsolicited inbound connections. A NAT Gateway achieves exactly this — outbound only.

**Why NSG over relying on VM firewalls alone?**
NSGs enforce access control at the network level, before traffic even reaches the VM. This adds a layer of defense independent of the OS configuration.

---

## 🔁 Traffic Flow

**User → Application**
```
Browser → HTTPS (443) → OwnCloud VM (Public IP) → MySQL VM (Port 3306, Private IP)
```

**Admin → DB Server**
```
Admin → SSH (22) → App Server (Public IP) → SSH → MySQL VM (Private IP)
```

**DB Server → Internet (Outbound Only)**
```
MySQL VM → NAT Gateway → Internet (package updates, patches)
```

---

## 📸 Screenshots

### Virtual Network & Subnet Setup
| | |
|---|---|
| ![VNet Creation](screenshots/01_Vnet_creation.png) | ![Public Subnet](screenshots/02_Public_subnet_creation.png) |
| VNet Creation | Public Subnet Configuration |
| ![Private Subnet](screenshots/03_Private_subnet_creation.png) | |
| Private Subnet Configuration | |

### App Server (OwnCloud)
| | |
|---|---|
| ![App Server Creation](screenshots/04_Appserver_creation.png) | ![App Server Overview](screenshots/05_Appserver_creation_contd.png) |
| App Server VM Creation | App Server Overview |
| ![App Server NSG](screenshots/06_Appserver_security_group.png) | ![OwnCloud Install](screenshots/13_Appserver_owncloud_install.png) |
| App Server Security Group Rules | OwnCloud Installation |
| ![OwnCloud Scripts](screenshots/14_Appserver_owncloud_scripts.png) | |
| OwnCloud Setup Scripts | |

### DB Server (MySQL)
| | |
|---|---|
| ![DB Server Creation](screenshots/07_Dbserver_creation.png) | ![DB Server Overview](screenshots/08_Dbserver_creation_contd.png) |
| DB Server VM Creation | DB Server Overview |
| ![DB NSG](screenshots/09_Dbserver_security_group.png) | ![MySQL Install](screenshots/10_Dbserver_mysql_install.png) |
| DB Server Security Group Rules | MySQL Installation |
| ![MySQL Install contd](screenshots/11_Dbserver_mysql_install_contd.png) | ![MySQL Config](screenshots/12_Dbserver_mysql_install_final.png) |
| MySQL Installation (contd.) | MySQL Configuration Complete |

### Final Verification
![OwnCloud Browser](screenshots/15_Accessing_owncloud_browser.png)
*OwnCloud successfully accessed via the App Server's public IP in a browser*

---

## 📁 Repository Structure

```
.
├── screenshots/
│   ├── 01_Vnet_creation.png
│   ├── 02_Public_subnet_creation.png
│   ├── 03_Private_subnet_creation.png
│   ├── 04_Appserver_creation.png
│   ├── 05_Appserver_creation_contd.png
│   ├── 06_Appserver_security_group.png
│   ├── 07_Dbserver_creation.png
│   ├── 08_Dbserver_creation_contd.png
│   ├── 09_Dbserver_security_group.png
│   ├── 10_Dbserver_mysql_install.png
│   ├── 11_Dbserver_mysql_install_contd.png
│   ├── 12_Dbserver_mysql_install_final.png
│   ├── 13_Appserver_owncloud_install.png
│   ├── 14_Appserver_owncloud_scripts.png
│   └── 15_Accessing_owncloud_browser.png
├── nsg-rules.md
├── architecture.png
└── README.md
```

---

## 💡 Key Learnings

- Hands-on experience designing and implementing **multi-tier network architecture** on a public cloud provider
- Applied **network segmentation** principles by isolating application and database layers
- Understood how **NSGs act as stateful firewalls** at the subnet level in Azure
- Learned the **Bastion Host access pattern** as a secure alternative to exposing SSH publicly
- Gained practical experience with **Linux server administration** — installing, configuring, and troubleshooting services on Ubuntu
- Understood the role of a **NAT Gateway** in enabling secure outbound-only connectivity for private resources

---

## 📄 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
