# **Complete Step-by-Step Guide to Set up Trap and Operator then Earning Cadet & Corporal Roles in Drosera Network (Hoodi Testnet)**

## **📌 Table of Contents**
1. [System Requirements](#-system-requirements)
2. [Getting Started](#-getting-started)
3. [VPS Setup](#-vps-setup)
4. [Trap Deployment](#-trap-deployment)
5. [Operator Setup](#-operator-setup)
   - [Method 1: Docker](#method-1-docker)
   - [Method 2: SystemD](#method-2-systemd)
6. [Final Verification](#-final-verification)
7. [Troubleshooting](#-troubleshooting)

---

## **🖥️ System Requirements**
- **Minimum VPS Specs**:
  - 4GB RAM
  - 2 CPU Cores
  - 20GB SSD Storage


---

## **🚀 Getting Started**
### **1. Get Testnet ETH (Faucets)**
You'll need Hoodi ETH for gas fees:
- 🔨 **Mining Faucet**: [https://hoodi-faucet.pk910.de](https://hoodi-faucet.pk910.de)
- 💧 **Public Faucets**:
  - [QuickNode Faucet](https://faucet.quicknode.com/hoodi)
  - [Stakely Faucet](https://stakely.io/faucet/ethereum-hoodi-testnet-eth)

### **2. Get Hoodi RPC**
- Visit [Ankr RPC](https://www.ankr.com/rpc/projects) or [Alchemy](https://dashboard.alchemy.com/)
- Switch to Hoodi testnet and copy your RPC URL

---

### **2. Update System**
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### **3. Install Dependencies**
```bash
sudo apt install -y curl ufw iptables build-essential git wget lz4 jq make gcc nano \
automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev \
tar clang bsdmainutils ncdu unzip
```

---

## **🕸️ Trap Deployment**
### **1. Install Required Tools**
```bash
# Drosera CLI
curl -L https://app.drosera.io/install | bash
source ~/.bashrc
droseraup
```
```bash
# Foundry
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup
```
```bash
# Bun JS
curl -fsSL https://bun.sh/install | bash
source ~/.bashrc
```

### **2. Create and Deploy Trap**
```bash
mkdir my-drosera-trap && cd my-drosera-trap
git config --global user.email "you@example.com"
git config --global user.name "your_username"
```

#### **Edit Trap.sol**
```bash
nano src/Trap.sol
```
Paste this (replace `YOUR-DISCORD-USERNAME`):
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IMockResponse {
    function isActive() external view returns (bool);
}

contract Trap is ITrap {
    address public constant RESPONSE_CONTRACT = 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608;
    string constant discordName = "YOUR-DISCORD-USERNAME";

    function collect() external view returns (bytes memory) {
        bool active = IMockResponse(RESPONSE_CONTRACT).isActive();
        return abi.encode(active, discordName);
    }

    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory) {
        (bool active, string memory name) = abi.decode(data[0], (bool, string));
        if (!active || bytes(name).length == 0) {
            return (false, bytes(""));
        }
        return (true, abi.encode(name));
    }
}
```
- Change `YOUR-DISCORD-USERNAME` to your actual Discord Username

- Save File by Pressing `Ctrl + X` then `Y` then Click `Enter`



### **3. Initialize**
```bash
forge init -t drosera-network/trap-foundry-template
```
- Install JS dependencies and compile:
```bash
bun install

forge build
```
### ***4. Edit Drosera.toml file***
```bash
nano drosera.toml
```
Update the variables with the following values:
`ethereum_rpc = "YOUR_ANKR/ALCHEMY_PRIVATE_HOODI_RPC"`

`path = "out/Trap.sol/Trap.json"`

`response_contract = "0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608"`

`response_function = "respondWithDiscordName(string)"`

### **5. Deploy**
```bash
DROSERA_PRIVATE_KEY=YOUR_PRIVATE_KEY drosera apply
```
- Replace `YOUR_PRIVATE_KEY` with your Private keys.
- When prompted, type `"ofc"` and hit Enter


### **6. Verify on Dashboard**
- Visit [Drosera App](https://app.drosera.io)
- Connect wallet and check "Traps Owned"


## **⚙️ Operator Setup**

### **Method 1: Docker Setup (Recommended)**
#### **1. Create Project Folder**
```bash
cd ~
mkdir Drosera-Network
cd Drosera-Network
```

#### **2. Create `docker-compose.yaml`**
```bash
nano docker-compose.yaml
```
Paste this configuration (**do not put private keys/IPs here**) only replace public with your RPC from ANKR/ALCHEMY :
```yaml
version: '3.8'

services:
  drosera-operator:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-operator
    ports:
      - "31313:31313"   # P2P
      - "31314:31314"   # HTTP
    environment:
      - DRO__DB_FILE_PATH=/data/drosera.db
      - DRO__DROSERA_ADDRESS=0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
      - DRO__LISTEN_ADDRESS=0.0.0.0
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__BACKUP_RPC_URL=https://rpc.hoodi.ethpandaops.io
      - DRO__ETH__PRIVATE_KEY=${ETH_PRIVATE_KEY}
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${VPS_IP}
      - DRO__SERVER__PORT=31314
      - RUST_LOG=info,drosera_operator=debug
    volumes:
      - drosera_data:/data
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:31314/health"]
      interval: 60s
      timeout: 10s
      retries: 3
    command: node

volumes:
  drosera_data:
```

#### **3. Create `.env` File**
```bash
nano .env
```
Add your private key and VPS IP:
```ini
ETH_PRIVATE_KEY=your_private_key_here
VPS_IP=your_vps_public_ip_here
```

#### **4. Pull Docker Image & Start**
```bash
docker pull ghcr.io/drosera-network/drosera-operator:latest
docker compose down -v  # Clean old volumes
docker compose up -d
docker compose logs -f  # View live logs
```

---

### **Method 2: SystemD Setup (Advanced)**
#### **1. Remove Old Service (If Exists)**
```bash
sudo systemctl stop drosera
sudo systemctl disable drosera
sudo rm /etc/systemd/system/drosera.service
sudo systemctl daemon-reload
```

#### **2. Create SystemD Service**
Replace `PV_KEY` (private key) and `VPS_IP` (public IP):
```bash
sudo tee /etc/systemd/system/drosera.service > /dev/null <<EOF
[Unit]
Description=drosera node service
After=network-online.target

[Service]
User=$USER
Restart=always
RestartSec=15
LimitNOFILE=65535
ExecStart=$(which drosera-operator) node --db-file-path $HOME/.drosera.db --network-p2p-port 31313 --server-port 31314 \
    --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
    --eth-backup-rpc-url https://rpc.hoodi.ethpandaops.io \
    --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D \
    --eth-private-key PV_KEY \
    --listen-address 0.0.0.0 \
    --network-external-p2p-address VPS_IP \
    --disable-dnr-confirmation true \
    --eth-chain-id 560048

[Install]
WantedBy=multi-user.target
EOF
```

#### **3. Start Service**
```bash
sudo systemctl daemon-reload
sudo systemctl enable drosera
sudo systemctl start drosera
journalctl -u drosera.service -f  # Live logs
```

---

### **🔧 Post-Setup Steps (Both Methods)**
1. **Open Firewall Ports**:
   ```bash
   sudo ufw allow 31313/tcp
   sudo ufw allow 31314/tcp
   sudo ufw enable
   ```

2. **Verify Node Status**:
   - **Docker**: `docker ps` (check if container is running)  
   - **SystemD**: `sudo systemctl status drosera`  

3. **Opt-In on Dashboard**:
   - Visit [app.drosera.io](https://app.drosera.io) → Connect Wallet → Click **"Opt-In"**.

---

### **🚨 Troubleshooting**
- **Port Conflicts**: Ensure ports `31313-31314` are free (`sudo lsof -i :31313`).  
- **RPC Errors**: Switch to backup RPC in config.  
- **Logs Investigation**:  
  - Docker: `docker compose logs -f`  
  - SystemD: `journalctl -u drosera.service -f`  

---

**To Earn Corporal Role**
- Go to official Discord and open a ticket, then submit Green Blocks screenshot.

This integrates **both deployment methods** with security best practices (`.env` for secrets, health checks, and automatic restarts). Choose Docker for simplicity or SystemD for granular control.
