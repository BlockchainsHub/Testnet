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
wget -q https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz -O /tmp/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
tar -xzf /tmp/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz -C /tmp
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
sudo cp /tmp/geth-linux-amd64-0.9.2-ea9f0d2/geth $HOME/go/bin/story-geth
```

### 4. Download and Install Story Binary
```bash
wget -q https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz -O /tmp/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar -xzf /tmp/story-linux-amd64-0.9.11-2a25df1.tar.gz -C /tmp
sudo cp /tmp/story-linux-amd64-0.9.11-2a25df1/story $HOME/go/bin/story
```

### 5. Initialize The Iliad Network Node
```bash
$HOME/go/bin/story init --network iliad --moniker "your_moniker"
```

### 6. Create and Configure systemd Service for Story-Geth
```bash
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=$HOME/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 7. Create and Configure systemd Service for Story
```bash
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=$HOME/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 8. Reload systemd, Enable, and Start Services
```bash
sudo systemctl daemon-reload
sudo systemctl enable story-geth story
sudo systemctl start story-geth story
```

### 9. Check Service Status
Checking Story-Geth service status.
```bash
sudo systemctl status story-geth --no-pager -l
```
Checking Story service status.
```bash
sudo systemctl status story --no-pager -l
```

### 10. Check Logs
Checking logs for Story-Geth
```bash
sudo journalctl -u story-geth -f -o cat
```
Checking logs for Story
```bash
sudo journalctl -u story -f -o cat
```

### 11. Check Sync Status
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
https://faucet.story.foundation/
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