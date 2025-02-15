![Github Banner-3-Octra Github Banner](https://github.com/user-attachments/assets/326ef408-f080-4a83-9c0e-4761a4fca92e)

# Pipe PoP Cache Node Guide
## Overview
Cache Node is a high-performance caching service that helps distribute and serve files efficiently across a network.

## System Requirements
| System | Requirements |
|-|-
| Memory | Minimum 4GB (configurable) |
| Storage | Minimum 100GB (configurable) |
| OS | Linux (Ubuntu 22.04 or above) |

## Installation
### 1. Install Dependencies
```bash
sudo apt update
sudo apt install -y libssl3
```

### 2. Setup the directories
```bash
sudo mkdir -p /opt/pop
sudo mkdir -p /var/lib/pop
sudo mkdir -p /var/cache/pop/download_cache
```

### 3. Setup directories permission
```bash
sudo chown -R $USER:$USER /opt/pop
sudo chown -R $USER:$USER /var/lib/pop
sudo chown -R $USER:$USER /var/cache/pop
```

### 4. Donwload the compiled binary
```bash
cd $HOME
curl -L -o pop https://dl.pipecdn.app/v0.2.5/pop
chmod +x pop
sudo mv pop /opt/pop
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

### 8. Create systemd service file
```bash
sudo tee /etc/systemd/system/pop.service << 'EOF'
[Unit]
Description=Pipe POP Node Service
After=network.target
Wants=network-online.target

[Service]
User=$USER
Group=$USER
ExecStart=/opt/pop/pop \\
    --ram $ram_size \\
    --pubKey $solana_address \\
    --max-disk $storage_size \\
    --cache-dir /var/cache/pop/download_cache \\
    --no-prompt
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

### 9. Set config file symlink and pop alias
Set the symlink and pop alias to avoid duplicated configs or registrations.
```bash
ln -sf /var/lib/pop/node_info.json ~/node_info.json
grep -q "alias pop='cd /var/lib/pop && /opt/pop/pop'" ~/.bashrc || echo "alias pop='cd /var/lib/pop && /opt/pop/pop'" >> ~/.bashrc && source ~/.bashrc
```

### 10. Reload systemd, check and enable service
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
