![Story Protocol Github Banner](https://github.com/user-attachments/assets/51fe2eef-9e7d-4550-9b85-870c4e98117c)
# Story Node Guide
This guide will assist you in the installation process of the Story node.

-----------------------------------------------------------------

## Required Specifications
| Required | Specifications |
|-|-
| CPU | 4 Cores |
| Memory | 8 GB |
| Storage | 200 GB |
| Bandwidth | 10 MBit/s |
| OS | Linux |

-----------------------------------------------------------------

## Node Installation
### 1. Install Required Packages
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

### 2. Install Go
```bash
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

### 3. Download and Install Story-Geth Binary
```bash
wget -q -O geth https://github.com/piplabs/story-geth/releases/download/v0.10.0/geth-linux-amd64
wget -q -O geth.sha256 https://github.com/piplabs/story-geth/releases/download/v0.10.0/geth-linux-amd64.sha256
sha256sum -c geth.sha256
chmod +x geth
sudo mv geth ~/go/bin/geth
```

### 4. Download and Install Story Binary
```bash
wget -q -O story https://github.com/piplabs/story/releases/download/v0.12.0/story-linux-amd64
chmod +x story
mkdir -p ~/go/bin
mkdir -p ~/.story/story/cosmovisor/genesis/bin
sudo mv story ~/go/bin/story
sudo cp ~/go/bin/story ~/.story/story/cosmovisor/genesis/bin
```

### 5. Install and Setup The Latest Version of Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
mkdir -p ~/.story/story/cosmovisor
echo "export DAEMON_NAME=story" >> ~/.bash_profile
echo "export DAEMON_HOME=$HOME/.story/story" >> ~/.bash_profile
echo "export PATH=$HOME/go/bin:$DAEMON_HOME/cosmovisor/current/bin:$PATH" >> ~/.bash_profile
source ~/.bash_profile
```

### 6. Initialize The Odyssey Network Node
```bash
~/.story/story/cosmovisor/genesis/bin/story init --network odyssey --moniker "your_moniker"
```

### 7. Update Peers
```
PEERS=$(curl -sS https://story-testnet-rpc.blockhub.id | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}' | paste -sd, -)
echo "Updating peers..."
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.story/story/config/config.toml
```

### 8. Create and Configure systemd Service for Geth
```bash
sudo tee /etc/systemd/system/geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/geth --odyssey --syncmode full --http --http.api eth,net,web3,engine --http.vhosts '*' --http.addr 0.0.0.0 --http.port 8545 --ws --ws.api eth,web3,net,txpool --ws.addr 0.0.0.0 --ws.port 8546
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 9. Create and Configure systemd Service for Story
```bash
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Cosmovisor service for Story binary
After=network.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run run
WorkingDirectory=$HOME/.story/story
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
Environment="DAEMON_NAME=story"
Environment="DAEMON_HOME=$HOME/.story/story"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_DATA_BACKUP_DIR=$HOME/.story/story/data"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
EOF
```

### 10. Reload systemd, Enable, and Start Services
```bash
sudo systemctl daemon-reload
sudo systemctl enable geth story
sudo systemctl start geth story
```

### 11. Check Service Status
Checking Geth service status.
```bash
sudo systemctl status geth --no-pager -l
```
Checking Story service status.
```bash
sudo systemctl status story --no-pager -l
```

### 12. Check Logs
Checking logs for Geth
```bash
sudo journalctl -u geth -f -o cat
```
Checking logs for Story
```bash
sudo journalctl -u story -f -o cat
```

### 13. Check Sync Status
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
story validator export --export-evm-key
```
> [!CAUTION]
> Backup your Public Key and Private Key.

### 2. Request Faucet
To create your validator you need at least 1 IP (1000000000000000000) to be staked. You can get 1 IP per day from the faucet.

<a href="https://faucet.story.foundation/" target="_blank">
  <img src="https://github.com/user-attachments/assets/b76fa86a-6c14-4c64-b96f-743d1fe4b73e" alt="Story Faucet" width="150" border="10" />
</a>

> [!NOTE]
> You need more than 1 IP because you have to pay for the fee.

### 3. Create Validator
```bash
story validator create --stake 1000000000000000000 --private-key "your_private_key"
```
> [!CAUTION]
> Don't forget to backup the `priv_validator_key.json` file located in `$HOME/.0gchain/config/`.

-----------------------------------------------------------------

# Command List
### Check Validator Public Key
```bash
curl -s localhost:26657/status | jq -r '.result.validator_info.pub_key.value'
```

### Delegate Tokens to Your Validator
Change `$VALIDATOR_PUB_KEY_IN_BASE64` with your validator public key and `$PRIVATE_KEY` with your private key.
```bash
story validator stake \
   --validator-pubkey "$VALIDATOR_PUB_KEY_IN_BASE64" \
   --stake 1000000000000000000 \
   --private-key $PRIVATE_KEY
```

### Check Validator HEX Address
You can copy and paste your validator HEX address on the [explorer](https://testnet.story.explorers.guru/) search bar to see your validator.
```bash
curl -s localhost:26657/status | jq -r '.result.validator_info.address'
```

### Remove Node
> [!CAUTION]
> Before remove the node **make sure you have already backup all the important data!**
```bash
sudo systemctl stop geth
sudo systemctl stop story
sudo systemctl disable geth
sudo systemctl disable story
sudo rm /etc/systemd/system/geth.service
sudo rm /etc/systemd/system/story.service
sudo systemctl daemon-reload
sudo rm -rf $HOME/.story
sudo rm $HOME/go/bin/geth
sudo rm $HOME/go/bin/story
```
