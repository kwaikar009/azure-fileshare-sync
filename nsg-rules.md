# Network Security Group (NSG) Configuration

## Overview
NSGs are configured at both the **subnet level** and **NIC level** to enforce layered security across the architecture.

---

## App Layer NSG — Inbound Rules

| Priority | Name              | Port | Protocol | Source          | Action |
|----------|-------------------|------|----------|-----------------|--------|
| 100      | Allow-HTTPS       | 443  | TCP      | Internet        | Allow  |
| 110      | Allow-HTTP-Redirect | 80 | TCP      | Internet        | Allow  |
| 200      | Allow-SSH-Admin   | 22   | TCP      | AdminIP/32      | Allow  |
| 4096     | Deny-All-Inbound  | *    | *        | *               | Deny   |

## App Layer NSG — Outbound Rules

| Priority | Name              | Port | Protocol | Destination     | Action |
|----------|-------------------|------|----------|-----------------|--------|
| 100      | Allow-MySQL       | 3306 | TCP      | DB Subnet       | Allow  |
| 110      | Allow-BlobStorage | 443  | TCP      | Storage Account | Allow  |
| 4096     | Deny-All-Outbound | *    | *        | *               | Deny   |

---

## Database Layer NSG — Inbound Rules

| Priority | Name              | Port | Protocol | Source          | Action |
|----------|-------------------|------|----------|-----------------|--------|
| 100      | Allow-MySQL-App   | 3306 | TCP      | App Subnet      | Allow  |
| 4096     | Deny-All-Inbound  | *    | *        | *               | Deny   |

> ⚠️ MySQL port 3306 is **only accessible from within the VNet App Subnet** — never exposed to public internet.

---

## Key Security Principles Applied
- **Default Deny**: All traffic denied unless explicitly allowed
- **Least Privilege**: Each layer only communicates with the layer directly below/above it
- **Admin Access**: SSH restricted to a single trusted IP
