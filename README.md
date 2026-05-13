# OwnCloud Deployment on Azure — Secure Two-Tier Architecture

![Azure](https://img.shields.io/badge/Cloud-Microsoft%20Azure-0078D4?logo=microsoftazure&logoColor=white)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu-E95420?logo=ubuntu&logoColor=white)
![MySQL](https://img.shields.io/badge/Database-MySQL-4479A1?logo=mysql&logoColor=white)
![OwnCloud](https://img.shields.io/badge/App-OwnCloud-0082C9?logo=owncloud&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

This project demonstrates the deployment of a secure, production-style two-tier architecture on Microsoft Azure using OwnCloud as a self-hosted file sharing platform.

The architecture implements strict network segmentation by separating application and database tiers into public and private subnets, enforcing secure communication patterns, and minimizing external exposure through Azure networking services.

---

## 📐 Architecture Overview

![Architecture Diagram](architecture.png)

The solution is built using a two-tier architecture:
- **Application Tier:**: OwnCloud hosted on an Ubuntu VM in a public subnet
- **Database Tier:**: MySQL hosted on an Ubuntu VM in a private subnet with no public exposure
    
Key design principles:

  - Network isolation between application and database layers
  - Controlled access via NSGs
  - Secure administrative access using bastion-style SSH pattern
  - Outbound-only internet access for private resources via NAT Gateway
---

## 🛠️ Azure Services Used

| Layer | Technology |
|---|---|
| Cloud Platform | Microsoft Azure |
| Virtual Network | Azure VNet, Public & Private Subnets |
| Network Security | Network Security Groups (NSG) |
| Outbound Connectivity | Azure NAT Gateway |
| Secure Administration | Bastion Host (SSH/RDP) |
| Application Server | OwnCloud on Ubuntu 22.04 |
| Database Server | MySQL on Ubuntu 22.04 |
| Operating System | Ubuntu Linux 22.04 LTS |

---

## 🔐 Security Architecture

- Database tier deployed in a private subnet with no public IP exposure  
- NSG-based traffic filtering enforcing least-privilege access principles  
- Bastion-style access using the application VM as a jump host  
- NAT Gateway enabling secure outbound-only internet access for private VM updates  
- Direct inbound internet access to the database tier is fully eliminated  
---

## 🔁 High-Level Traffic Flow

**Application Access**
```
User → HTTPS (443) → OwnCloud Application VM (Public IP) → MySQL Database VM (Port 3306, Private IP)
```

**Administrative Access (Bastion Pattern)**
```
Admin → SSH (22) → Application Server (Public IP) → SSH → Database Server (Private IP)
```

**Outbound Updates**
```
Database Server → NAT Gateway → Internet (package updates, patches)
```

---

## 🔒 Security Design Decisions

- **Why a Private Subnet for Database?**  
  To eliminate direct internet exposure and reduce attack surface.

- **Why bastion-style access?**  
  To avoid exposing SSH access directly to database servers.

- **Why a NAT Gateway?**  
  To allow secure outbound connectivity without enabling inbound access.

- **Why NSG over relying on VM firewalls alone?**  
  To enforce network-level security controls independent of OS configuration.

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

- Designing secure multi-tier cloud architectures
- Implementing subnet-level isolation in Azure
- Understanding real-world traffic flow patterns
- Practical Linux-based server provisioning and configuration
- Applying cloud security best practices (least privilege, segmentation, controlled access)

---

## 🚀 Future Improvements

- Automate deployment using Terraform (Infrastructure as Code)
- Replace MySQL VM with Azure Database for MySQL (managed service)
- Add Azure Load Balancer for scalability
- Implement CI/CD pipeline for infrastructure provisioning
- Add centralized logging/monitoring (Azure Monitor)

