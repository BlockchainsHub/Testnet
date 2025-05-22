![Github Banner-3-Octra Github Banner](https://github.com/user-attachments/assets/326ef408-f080-4a83-9c0e-4761a4fca92e)

# Pipe PoP Cache Testnet Node Guide
## Overview
Cache Node is a high-performance caching service that helps distribute and serve files efficiently across a network.

## System Requirements
| System | Requirements |
|-|-
| CPU | 4+ Cores |
| Memory | 16GB+ |
| Storage | 100GB+ SSD |
| Bandwidth | 1Gbps+ |
| OS | Linux (Ubuntu 22.04 or above) |

## Installation
### 1. Install dependencies
```bash
sudo apt update
sudo apt install -y libssl-dev ca-certificates
```

### 2. Optimize system settings for network performance
```bash
sudo bash -c 'cat > /etc/sysctl.d/99-popcache.conf << EOL
net.ipv4.ip_local_port_range = 1024 65535
net.core.somaxconn = 65535
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.core.wmem_max = 16777216
net.core.rmem_max = 16777216
EOL'

sudo sysctl -p /etc/sysctl.d/99-popcache.conf
```

### 3. Increase file limits for performance
```bash
sudo bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
*    hard nofile 65535
*    soft nofile 65535
EOL'
```

### 2. Setup the directories and create configuration file
```bash
sudo mkdir -p /opt/popcache
cd /opt/popcache
sudo nano /opt/popcache/config.toml
```
Insert the following configuration, read the configuration tips below and adjust the values as needed:
```bash
{
  "pop_name": "your-pop-name",
  "pop_location": "Your Server Location, Country",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 40
  },
  "cache_config": {
    "memory_cache_size_mb": 4096, 
    "disk_cache_path": "./cache",
    "disk_cache_size_gb": 100,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "your-node-name",
    "name": "Your Name",
    "email": "your.email@example.com",
    "website": "https://your-website.com",
    "discord": "your_discord_username",
    "telegram": "your_telegram_handle",
    "solana_pubkey": "YOUR_SOLANA_WALLET_ADDRESS_FOR_REWARDS"
  }
}
```

| Configuration | Tips |
|-|-
| `memory_cache_size_mb` | Set based on your available RAM (50-70% of total RAM is a good starting point) |
| `disk_cache_size_gb` | Set based on your available disk space (leave at least 20% free). Example: For a 500GB disk, set to 350-400 |



### 3. Setup directories permission
```bash
sudo chown -R $USER:$USER /opt/popcache
```

### 4. Donwload the compiled binary
Go to [Pop Client Download Portal](https://download.pipe.network/) and enter your invite code. Inside the portal choose the binary that you want to download and copy the link. Replace `$download_link` with the link you've copied.
```bash
curl $download_link
chmod +x pop
```

### 5. Register your node
```bash
cd /opt/pop/
./pop --signup-by-referral-route ae7dc44a4ba6bfc5
```

### 6. Handle any existing node_info.json from quick-start
```bash
sudo mv -f /opt/pop/node_info.json /var/lib/pop/ 2>/dev/null || true
```

### 7. Setup node configuration
Enter your RAM Allocation
```bash
read -p "Enter RAM allocation in GB (minimum 4GB): " ram_size
```
Enter your Solana address
```bash
read -p "Enter your Solana address: " solana_address
```

Enter your max storage allocation
```bash
read -p "Enter max storage allocation in GB (minimum 100GB): " storage_size
```

### 8. Enable firewall
```bash
sudo ufw allow ssh/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8003/tcp
sudo ufw enable
```

### 9. Create systemd service file
```bash
sudo tee /etc/systemd/system/pop.service << EOF
[Unit]
Description=Pipe POP Node Service
After=network.target
Wants=network-online.target

[Service]
User=${USER}
Group=${USER}
ExecStart=/opt/pop/pop \
    --ram ${ram_size} \
    --pubKey ${solana_address} \
    --max-disk ${storage_size} \
    --cache-dir /var/cache/pop/download_cache \
    --no-prompt
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
Restart=always
RestartSec=5
LimitNOFILE=65536
LimitNPROC=4096
StandardOutput=journal
StandardError=journal
SyslogIdentifier=pop-node
WorkingDirectory=/var/lib/pop

[Install]
WantedBy=multi-user.target
EOF
```

### 10. Set config file symlink and pop alias
Set the symlink and pop alias to avoid duplicated configs or registrations.
```bash
ln -sf /var/lib/pop/node_info.json ~/node_info.json
grep -q "alias pop='cd /var/lib/pop && /opt/pop/pop'" ~/.bashrc || echo "alias pop='cd /var/lib/pop && /opt/pop/pop'" >> ~/.bashrc && source ~/.bashrc
```

### 11. Reload systemd, check and enable service
```bash
sudo systemctl daemon-reload
sudo systemd-analyze verify pop.service && sudo systemctl enable pop.service && sudo systemctl start pop.service
```

-----------------------------------------------------------------

## Upgrade Node
### Stop the node
```bash
cd $HOME
sudo systemctl stop pop
```
### Download the binary
Please replace `$download_url` with the download link.
```bash
curl -L -o pop $download_url
chmod +x pop
sudo mv pop /opt/pop
```
### Restart the node
```bash
sudo systemctl start pop
```

-----------------------------------------------------------------

## Command list
### View metrics
```bash
pop --status
```
### Check points
```bash
pop --points
```
### Generate referral
```bash
pop --gen-referral-route
```
### Use referral
```bash
pop --signup-by-referral-route $referral_code
```
### Check for upgrade
```bash
pop --refresh
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2025 BlockHub. All rights reserved.
</p>
