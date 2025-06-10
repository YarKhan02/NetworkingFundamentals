# How To Create a VPC on AWS

The IP address `10.200.123.0/24` represents a **CIDR (Classless Inter-Domain Routing) block**, defining the **IP address range** for your **VPC (Virtual Private Cloud)**.

---

## CIDR Breakdown

- **Network Address**: `10.200.123.0` — reserved to identify the network.
- **Subnet Mask**: `/24` — means **24 bits** for network, **8 bits** for hosts.
- **Decimal Equivalent**: `255.255.255.0`
- **Total IPs**: 256  
- **Usable Host IPs**: 254  
  - **First usable**: `10.200.123.1`  
  - **Last usable**: `10.200.123.254`

---

## Subnets

Using a `/28` subnet mask for each subnet, which is a smaller division of the VPC range `10.200.123.0/24`.

### What Does `/28` Mean?

- **Total IP addresses**: 16
- Reserved:
  - 1 for network address
  - 1 for broadcast address
  - 3 reserved by AWS
- **Usable IPs**: 11

### Subnet Configuration

| Subnet Type | CIDR Block           | Usable IP Range             |
|-------------|----------------------|-----------------------------|
| Public      | `10.200.123.0/28`    | `10.200.123.4` - `10.200.123.14` |
| Private     | `10.200.123.128/28`  | `10.200.123.132` - `10.200.123.142` |

---

## Route Tables and Internet Gateway

### VPC Details

- **VPC CIDR**: `10.200.123.0/24`
- **Public Subnet**: `10.200.123.0/28`
- **Private Subnet**: `10.200.123.128/28`

### Public Subnet Route Table

Associated with `10.200.123.0/28`

| Destination       | Target                        | Purpose                              |
|-------------------|-------------------------------|--------------------------------------|
| `10.200.123.0/24` | `local`                       | Enables communication within VPC     |
| `0.0.0.0/0`       | `igw-0b72c54f9a66b3624`       | Enables outbound internet access     |

### Private Subnet Route Table

Associated with `10.200.123.128/28`

| Destination       | Target  | Purpose                          |
|-------------------|---------|----------------------------------|
| `10.200.123.0/24` | `local` | Enables communication within VPC|

> No `0.0.0.0/0` route here, so instances cannot reach the internet directly.

---

## Internet Gateway

- **ID**: `igw-0b72c54f9a66b3624`
- **Attached to the VPC**
- Enables bidirectional internet access for subnets routing `0.0.0.0/0` to it
- Only the public subnet can send/receive internet traffic; private subnet is internal only

---

## Security Group Configuration

### Inbound Rules

| Rule | Protocol | Port | Source              | Purpose                          |
|------|----------|------|---------------------|----------------------------------|
| 1    | SSH      | 22   | `182.178.107.201/32`| SSH access to Linux instances    |
| 2    | All      | All  | `10.200.123.0/24`   | Internal traffic within the VPC |
| 3    | RDP      | 3389 | `182.178.107.201/32`| RDP access to Windows instances  |

- Only your public IP can access instances over SSH or RDP.
- All internal VPC communication is allowed.
- All other inbound traffic is blocked.

### Outbound Rules

| Rule | Protocol | Port | Destination | Purpose                          |
|------|----------|------|-------------|----------------------------------|
| 1    | All      | All  | `0.0.0.0/0` | Allow all outbound traffic       |

---

## Key Pairs

Two key pairs were created:

1. `network-ppk.ppk` — for Ubuntu EC2 instance (SSH)
2. `network-pem.pem` — for Windows EC2 instance (RDP)

### `network-ppk.ppk` (Ubuntu)

- PuTTY Private Key
- Used with PuTTY or MobaXterm for SSH into Ubuntu instance

### `network-pem.pem` (Windows)

- PEM key used to decrypt the Windows administrator password in AWS Console
- After decryption, use the password for RDP access

---

## Network Interfaces (ENIs)

Two ENIs were created and configured:

- Attached to Ubuntu and Windows EC2 instances
- Placed in the private subnet (`10.200.123.128/28`)
- Assigned appropriate security group
- Automatically linked to the VPC when assigned to subnet

### Technical Details

| Component        | Configuration                                               |
|------------------|-------------------------------------------------------------|
| ENI              | Virtual network interface in private subnet                |
| Subnet           | `10.200.123.128/28` with no direct internet access         |
| Security Group   | Allows SSH (22), RDP (3389) from your IP                   |
| Route Table      | Only local route, no NAT or IGW                            |
| Result           | EC2s are isolated, only accessible internally              |

---

## Purpose of Two Network Interfaces

### Public Network Interface (in Public Subnet)

- Has public IP or is connected to Internet Gateway
- Used for external access:
  - SSH (Ubuntu)
  - RDP (Windows)
  - Software updates and internet access
- Security group limits exposure

### Private Network Interface (in Private Subnet)

- Only private IP, no internet access
- Used for internal communication:
  - EC2 to EC2
  - EC2 to database or other internal services
- More secure, no public exposure
