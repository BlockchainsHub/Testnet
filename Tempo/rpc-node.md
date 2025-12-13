# Tempo Node Guide

## Introduction

Welcome to the BlockHub guide for setting up a **Tempo** node.

Tempo is a **payments-first blockchain** incubated by **Stripe and Paradigm**. Built on the **Reth** (Rust Ethereum) stack, it is engineered for 100,000+ TPS and sub-second finality, bridging the gap between stablecoins and real-world financial infrastructure.

By running a node, you are supporting a network designed for global payouts, on-chain FX, and compliant decentralized finance. This guide adopts a "Glass Box" approach: we will transparently explain the *why* behind every command, ensuring you understand the high-performance Rust infrastructure you are deploying.

## Hardware Requirements

Tempo is a high-throughput network. Ensure your dedicated server or VPS meets these specifications to ensure high uptime and sync speed.

| Component | Minimum (RPC) | Recommended (RPC) | Recommended (Validator) |
| :--- | :--- | :--- | :--- |
| **CPU** | 8 Cores | 16+ Cores | 16+ Cores |
| **RAM** | 16 GB | 32 GB | 32 GB |
| **Storage** | 250 GB SSD | 500 GB NVMe | 1 TB NVMe |
| **Network** | 500 Mbps | 1 Gbps | 1 Gbps |

## Installation Process

### 1\. System Preparation

First, we update our package lists and upgrade existing packages to ensure our server is secure and up-to-date. We also install essential tools like `curl` and `git` that are required for the installation.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git jq lz4 build-essential -y
```

### 2\. Firewall Configuration

Tempo uses specific ports for P2P communication and RPC interfaces. We will allow these through our firewall (`ufw`).

**Crucially, we will add comments to each rule.** This is a best practice that ensures when you review your firewall settings in the future, you know exactly what each open port is used for.

  * **30303 (TCP/UDP):** P2P Peering (Public).
  * **8545 (TCP):** HTTP RPC (Public/Private).
  * **9000 (TCP):** Metrics (Internal).

<!-- end list -->

```bash
# Allow P2P Peering for syncing
sudo ufw allow 30303/tcp comment "Tempo P2P Peering"
sudo ufw allow 30303/udp comment "Tempo P2P Discovery"

# Allow RPC (Caution: This exposes your node's API)
sudo ufw allow 8545/tcp comment "Tempo HTTP RPC"

# Enable the firewall
sudo ufw enable
```

### 3\. Binary Installation

We will install the Tempo binary using their official installer. This script detects your architecture and places the binary in your local user directory.

```bash
curl -L https://tempo.xyz/install | bash
```

**Verification:**
After installation, reload your shell configuration and verify the binary is accessible.

```bash
source ~/.bashrc
tempo --version
```

*You should see a version output confirming the installation.*

### 4\. Initialization & Data Directory

Instead of a complex initialization process, Tempo requires us to create a dedicated data directory. This is where the blockchain database (built on Reth) will reside.

```bash
# Create the directory structure
mkdir -p $HOME/.tempo/data
```

### 5\. Snapshot Download (Genesis & State)

To speed up the synchronization process, Tempo provides a `download` command. This fetches the latest snapshot (genesis block and recent state), saving you days of syncing time compared to starting from block 0.

```bash
# Download and extract the latest snapshot
tempo download
```

**Why do we do this?**
Syncing from "Genesis" (the first block) involves replaying every transaction in history. A snapshot allows your node to jump ahead to a recent, trusted block height.

## Service File Configuration

We will use `systemd` to manage the Tempo node. This ensures the node runs in the background, restarts automatically if it crashes, and starts up when the server reboots.

We will configure the node with the following flags:

  * `--follow`: Follows the chain tip.
  * `--http`: Enables the HTTP RPC server.
  * `--discovery.addr`: Allows the node to be found by peers.

**Note on Paths:** We are explicitly pointing `ExecStart` to `$HOME/.tempo/bin/tempo`, which is where the installer placed the binary.

```bash
sudo tee /etc/systemd/system/tempo.service > /dev/null <<EOF
[Unit]
Description=Tempo RPC Node
After=network.target
Wants=network.target
 
[Service]
Type=simple
User=$USER
Group=$USER
Environment=RUST_LOG=info
WorkingDirectory=$HOME/.tempo
# Using the binary located in the user's local .tempo directory
ExecStart=$HOME/.tempo/bin/tempo node \\
  --datadir $HOME/.tempo/data \\
  --follow \\
  --port 30303 \\
  --discovery.addr 0.0.0.0 \\
  --discovery.port 30303 \\
  --http \\
  --http.addr 0.0.0.0 \\
  --http.port 8545 \\
  --http.api eth,net,web3,txpool,trace \\
  --metrics 9000 \\
 
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=tempo
LimitNOFILE=infinity
 
[Install]
WantedBy=multi-user.target
EOF
```

### Enable and Start

Now, reload the system daemon to recognize the new service, enable it to start on boot, and start the node immediately.

```bash
sudo systemctl daemon-reload
sudo systemctl enable tempo
sudo systemctl start tempo
```

## Command List (Cheat Sheet)

Here are the essential commands for managing your Tempo node.

### Log Monitoring

View the real-time logs of your node. This is your primary tool for debugging. Because Tempo is built on Reth/Rust, the logs are highly detailed.

```bash
# View real-time logs
sudo journalctl -u tempo -f -o cat
```

**Expected Output (Healthy Sync):**
You should see a rapid stream of `Block added to canonical chain` and `Forkchoice updated` messages. This confirms your node is keeping up with the sub-second block times.

```text
INFO Block added to canonical chain number=5223413 hash=0xa786... peers=6 txs=91
INFO Forkchoice updated head_block_hash=0xa786...
INFO Block added to canonical chain number=5223414 hash=0xc056... peers=6 txs=99
```

### Service Management

```bash
# Stop the node
sudo systemctl stop tempo

# Restart the node
sudo systemctl restart tempo
```

### Node Removal

[\!CAUTION]
**Danger Zone:** This will permanently delete your node data and configuration. Ensure you have backed up any necessary keys before running this.

```bash
sudo systemctl stop tempo
sudo systemctl disable tempo
rm /etc/systemd/system/tempo.service
rm -rf $HOME/.tempo
sudo systemctl daemon-reload
```

-----

<p align="center">
  &copy; <strong>BlockHub 2025</strong> | <em>Infrastructure for the Decentralized Future</em>
</p>