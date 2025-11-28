# Ubuntu Server Setup

This document describes how I created and configured the **Ubuntu Server (ARM)** virtual machine that serves as the main backend system of my cybersecurity home lab. I built this VM on my macOS M1 machine using UTM, and this setup will later host Docker-based Wazuh SIEM services, log collectors, and analysis tools.

---

## 📌 What I Wanted to Achieve

My goal with this step was to prepare a clean, stable, ARM-native Ubuntu Server where I can later deploy:
- Docker + Docker Compose  
- Wazuh Manager, Indexer and Dashboard  
- Sysmon for Linux  
- Custom detection rules  
- Future monitoring or IR tooling

This VM is the **foundation** of the entire lab.

---

## 🖥️ VM Configuration (UTM)

| Setting      | My Configuration        |
|--------------|--------------------------|
| Architecture | ARM64 (aarch64)         |
| RAM          | 2 GB                    |
| CPU          | 2–4 cores (I used 2)    |
| Disk         | 30–40 GB (I used 30 GB) |
| Network      | Shared (NAT)            |
| ISO          | Ubuntu Server 22.04 ARM |

### 📸 Screenshot suggestion:
- **UTM VM configuration screen**  
Place it later in: `/screenshots/utm-ubuntu-config.png`

---

## 🔧 Step-by-Step What I Did

### 1️⃣ Downloaded the Ubuntu Server ARM ISO

Official download page:  
https://ubuntu.com/download/server/arm

### 2️⃣ Created the VM in UTM

Inside UTM → Create New → Virtualize  
I configured:
- ARM architecture  
- 2 GB RAM  
- 2 vCPU  
- 30 GB disk  
- Shared Network (NAT)

### 3️⃣ Ubuntu Installation

During installation I selected:
- **OpenSSH Server**
- Hostname: `siem-ubuntu`
- User: `adam`

### 📸 Screenshot suggestion:
- Ubuntu installation finished screen  
- First terminal login  
Place them in `/screenshots/`

---

## 🔄 First Update After Installation

After logging in, I updated the system:

    sudo apt update && sudo apt upgrade -y

### 📸 Screenshot suggestion:
- Terminal showing successful upgrade  
Save as: `/screenshots/update-success.png`

---

## 🐳 Installing Docker & Docker Compose

I installed Docker on ARM using:

    sudo apt install docker.io docker-compose -y
    sudo systemctl enable docker
    sudo systemctl start docker

I verified the installation:

    docker --version
    docker compose version

### 📸 Screenshot suggestion:
- Output of docker --version  
- Output of docker ps (even if empty)  
Store as:  
`/screenshots/docker-version.png`  
`/screenshots/docker-ps.png`

---

## ✔️ Final State (What I Achieved)

By the end of this setup:
- Ubuntu Server runs smoothly on ARM  
- SSH access is available  
- Docker + Docker Compose work correctly  
- The system is ready for Wazuh SIEM deployment  
- I have a clean, stable base OS for later cybersecurity projects  

---

## 📝 Personal Notes / Things I Observed

(Add your thoughts here after you finish.)

Examples:
- How smooth the installation was  
- How UTM performed on 8 GB RAM  
- Any unexpected behavior  
- How Docker installation felt on ARM  
