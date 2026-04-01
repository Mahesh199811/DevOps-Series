# ☁️ Phase 1 — AWS Foundation

> Part of the [DevOps End-to-End Project](../README.md) | Cloud-Native Application on AWS

---

## 📋 Overview

This phase sets up the core AWS cloud environment — IAM, networking (VPC), and a running EC2 instance with Docker. This is the foundation everything else builds on.

---

## ✅ Day 1 — IAM Setup, CLI Configuration & VPC

### 🎯 Goals
- Secure the AWS root account
- Create a dedicated IAM user for daily work
- Configure AWS CLI on local machine
- Build the base network (VPC, subnets, internet gateway)

---

### 🔐 Step 1 — Secure Root Account

| Action | Details |
|--------|---------|
| Enable MFA on root | AWS Console → Security Credentials → MFA |
| MFA Type | Authenticator app (Google Authenticator / Authy) |
| Root usage going forward | ❌ Never use for daily work |

---

### 👤 Step 2 — IAM User Created

| Field | Value |
|-------|-------|
| Username | `devops-admin` |
| Access Type | Console + Programmatic |
| Permissions | `AdministratorAccess` (managed policy) |
| MFA on IAM user | ✅ Recommended |

> ⚠️ In production, always follow **least privilege** — AdministratorAccess is used here for learning only.

---

### 💻 Step 3 — AWS CLI Configured (macOS)

**Installation:**
```bash
brew install awscli
aws --version
```

**Configuration:**
```bash
aws configure
```

| Prompt | Value Used |
|--------|-----------|
| AWS Access Key ID | `[from IAM user credentials]` |
| AWS Secret Access Key | `[from IAM user credentials]` |
| Default region | `ap-south-1` (Mumbai) |
| Default output format | `json` |

**Verification:**
```bash
aws iam get-user
# Expected: returns devops-admin user details
```

---

### 🌐 Step 4 — VPC Created

**VPC Details:**

| Resource | Value |
|----------|-------|
| VPC Name | `devops-vpc` |
| IPv4 CIDR | `10.0.0.0/16` |
| Region | `ap-south-1` (Mumbai) |
| Public Subnets | 1 |
| Private Subnets | 1 |
| Internet Gateway | ✅ Attached |
| Route Tables | ✅ Auto-configured |
| NAT Gateway | ❌ Skipped (cost saving for learning) |

**Architecture:**
```
devops-vpc (10.0.0.0/16)
│
├── Public Subnet (10.0.1.0/24)
│   └── Route: 0.0.0.0/0 → Internet Gateway
│
└── Private Subnet (10.0.2.0/24)
    └── Route: local only
```

---

### ✅ Day 1 Checklist

- [x] Root account MFA enabled
- [x] IAM user `devops-admin` created
- [x] Logged in as IAM user (not root)
- [x] AWS CLI installed and configured
- [x] CLI verified with `aws iam get-user`
- [x] VPC `devops-vpc` created in Mumbai region

---

## ✅ Day 2 — EC2 Launch & Docker Installation

### 🎯 Goals
- Create a Key Pair for SSH access
- Configure a Security Group (firewall)
- Launch an Ubuntu EC2 instance in the public subnet
- SSH into the instance
- Install and verify Docker

---

### 🔑 Step 1 — Key Pair

| Field | Value |
|-------|-------|
| Name | `devops-key` |
| Type | RSA |
| Format | `.pem` (macOS) |
| Stored at | `~/.ssh/devops-key.pem` |

```bash
# Secure the key (required for SSH to work)
chmod 400 ~/.ssh/devops-key.pem
```

---

### 🛡️ Step 2 — Security Group

| Name | `devops-sg` |
|------|------------|
| VPC | `devops-vpc` |

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| SSH | TCP | 22 | My IP | Terminal access |
| Custom TCP | TCP | 8080 | 0.0.0.0/0 | App access |

**Outbound Rules:** All traffic allowed (default)

---

### 🖥️ Step 3 — EC2 Instance

| Field | Value |
|-------|-------|
| Name | `devops-server` |
| AMI | Ubuntu Server 24.04 LTS |
| Instance Type | `t2.micro` (Free Tier) |
| Key Pair | `devops-key` |
| VPC | `devops-vpc` |
| Subnet | Public Subnet |
| Auto-assign Public IP | ✅ Enabled |
| Security Group | `devops-sg` |
| Storage | 8 GB (default) |

---

### 🔌 Step 4 — SSH Connection

```bash
ssh -i ~/.ssh/devops-key.pem ubuntu@<PUBLIC-IP>
```

> Replace `<PUBLIC-IP>` with your EC2 instance's public IPv4 address from the console.

---

### 🐳 Step 5 — Docker Installation

```bash
# Update packages
sudo apt update

# Install Docker
sudo apt install docker.io -y

# Enable Docker to start on boot
sudo systemctl start docker
sudo systemctl enable docker

# Add ubuntu user to docker group
sudo usermod -aG docker ubuntu

# Apply group membership without re-login
newgrp docker

# Verify installation
docker --version
docker run hello-world
```

**Expected output:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

### ✅ Day 2 Checklist

- [x] Key pair `devops-key` created and secured (`chmod 400`)
- [x] Security Group `devops-sg` configured with SSH + port 8080
- [x] EC2 instance `devops-server` launched (Ubuntu 24.04, t2.micro)
- [x] Instance in public subnet with public IP assigned
- [x] SSH connection successful
- [x] Docker installed and `hello-world` container runs

---

## 📁 Repo Structure (Phase 1)

```
phase-1-aws/
├── README.md          ← This file
└── architecture.png   ← (Add VPC screenshot from AWS console)
```

---

## 🔗 Next Phase

👉 [Phase 2 — Application Setup (Dockerize .NET API)](../phase-2-app/README.md)

---

## 📌 Resources

- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Docker Install on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)