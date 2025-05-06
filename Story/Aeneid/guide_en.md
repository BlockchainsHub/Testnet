![Story Protocol Github Banner](https://github.com/user-attachments/assets/51fe2eef-9e7d-4550-9b85-870c4e98117c)
# Story Node Guide
This guide will assist you in the installation process of the Story node.

-----------------------------------------------------------------

## Required Specifications
| Required | Specifications |
|-|-
| CPU | 8 Cores |
| Memory | 32 GB |
| Storage | 500 GB NVMe |
| Bandwidth | 25 MBit/s |
| OS | Linux |

-----------------------------------------------------------------

## Node Installation
### 1. Install Required Packages
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 gh -y
```

### 2. Install Go
```bash
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bashrc && \
source ~/.bashrc && \
go version
```

### 3. Download and Install Story-Geth Binary
```bash
wget -q -O story-geth https://github.com/piplabs/story-geth/releases/download/v1.0.2/geth-linux-amd64
chmod +x story-geth
sudo mv story-geth $HOME/go/bin/story-geth
```

### 4. Create and Configure Systemd Service of Story-Geth
```bash
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/story-geth --aeneid --syncmode full --datadir=$HOME/.story/geth/story --authrpc.jwtsecret=$HOME/.story/geth/story/geth/jwtsecret
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
### 5. Start Story-Geth Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable story-geth
sudo systemctl start story-geth
```

### 6. Verify Story-Geth Service Status
```bash
sudo systemctl status story-geth --no-pager -l
```

### 7. Install and Setup The Latest Version of Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
echo "export DAEMON_NAME=story" >> $HOME/.bashrc
echo "export DAEMON_HOME=$HOME/.story/story" >> $HOME/.bashrc
echo "export DAEMON_DATA_BACKUP_DIR=${DAEMON_HOME}/cosmovisor/backup" >> $HOME/.bashrc
echo "export DAEMON_ALLOW_DOWNLOAD_BINARIES=false" >> $HOME/.bashrc
source $HOME/.bashrc

sudo mkdir -p \
  $DAEMON_HOME/cosmovisor/backup \
  $DAEMON_HOME/data
```

### 8. Download and Install Story Binary
```bash
wget -q -O story https://github.com/piplabs/story/releases/download/v1.2.0/story-linux-amd64
chmod +x story
sudo mv story $HOME/go/bin/story
```

### 9. Setup `.story` Directory Ownership
```bash
sudo chown -R $USER:$USER $HOME/.story
```

### 10. Init Story with Cosmovisor
Replace `$moniker_name` with your actual moniker name.
```bash
cosmovisor init story
cosmovisor run init --network aeneid --moniker $moniker_name
cosmovisor version
```

### 11. Create and Configure systemd Service for Story
```bash
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Cosmovisor service for Story binary
After=network.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run run --api-enable --api-address=0.0.0.0:1317
WorkingDirectory=$HOME/.story/story
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
Environment="DAEMON_NAME=story"
Environment="DAEMON_HOME=$HOME/.story/story"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_DATA_BACKUP_DIR=$HOME/.story/story/cosmovisor/backup"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
EOF
```

### 12. Start Story Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable story
sudo systemctl start story
```

### 13. Verify Story Service Status
```bash
sudo systemctl status story --no-pager -l
```

### 14. Check Logs
Checking logs for Story-Geth
```bash
sudo journalctl -u story-geth -fo cat
```
Checking logs for Story
```bash
sudo journalctl -u story -fo cat
```

### 15. Check Sync Status
If `catching_up` status is `true` that means that you are still syncing and if it's `false` it means that you are fully sync.
```bash
curl -s localhost:26657/status | jq
```
> [!CAUTION]
> Make sure your node is fully sync before continue to create validator.

-----------------------------------------------------------------

## Create Validator
### 1. Export Validator Public Key and Private Key
```bash
story validator export --export-evm-key --evm-key-path .env
```
> [!CAUTION]
> Backup your Public Key and Private Key.

### 2. Request Faucet
To create your validator you need at least 1024 IP (1024000000000000000000 wei) to be staked. You can fill this [form](https://docs.google.com/forms/d/e/1FAIpQLSdCgT7rSxGTkWXbBjzAZco8UyTBuTtjb87s5k0BEp5n6PF6jA/viewform) to get the tokens from the team. You can also get 10 from the faucet below.

<a href="https://cloud.google.com/application/web3/faucet/story/aeneid" target="_blank">
  <img src="https://github.com/user-attachments/assets/b76fa86a-6c14-4c64-b96f-743d1fe4b73e" alt="Story Faucet" width="150" border="10" />
</a>

> [!NOTE]
> You need more than 1024 IP because you have to pay for the fee.

### 3. Create Validator
Replace `$moniker_name` with your actual moniker name.
```bash
story validator create --stake 1024000000000000000000 --moniker $moniker_name --commission-rate 1000 --chain-id 1315 --explorer "https://aeneid.storyscan.io" --rpc "https://aeneid.storyrpc.io"
```
> [!CAUTION]
> Don't forget to backup the `priv_validator_key.json` file located in `$HOME/.story/story/config/`.

-----------------------------------------------------------------

# Command List

### Remove Node
> [!CAUTION]
> Before remove the node **make sure you have already backup all the important data!**
```bash
sudo systemctl stop story-geth
sudo systemctl stop story
sudo systemctl disable story-geth
sudo systemctl disable story
sudo rm /etc/systemd/system/story-geth.service
sudo rm /etc/systemd/system/story.service
sudo systemctl daemon-reload
sudo rm -rf $HOME/.story
sudo rm $HOME/go/bin/story-geth
sudo rm $HOME/go/bin/story
```
