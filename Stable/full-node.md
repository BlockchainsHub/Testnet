# Stable Node Guide
This guide will assist you in the installation process of the Stable node.

-----------------------------------------------------------------

## Hardware Requirements

| Component | Testnet                      |
| --------- | ---------------------------- |
| CPU       | 8 Cores                      |
| Memory    | 16 GB                        |
| Disk      | 1 TB NVMe SSD                |
| Bandwidth | 1 Gbps for Download / Upload |

-----------------------------------------------------------------

## Installation Process
### System Preparation
First, let's prepare the system with essential dependencies and updates:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git jq build-essential gcc unzip wget lz4 tmux aria2 pv
```

### Go Installation and Configuration
Go 1.22.0 is recommended for compatibility:

```bash
cd $HOME
ver="1.22.0"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
if ! grep -q 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' $HOME/.profile; then
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.profile
fi
mkdir -p $HOME/go/bin
go version
```

### Set Variables

```bash
echo "export MONIKER="node_name"" >> $HOME/.profile
source $HOME/.profile
```

### Stable Binary Installation

```bash
cd $HOME
mkdir -p stabled
cd stabled
wget https://stable-testnet-data.s3.us-east-1.amazonaws.com/stabled-latest-linux-amd64-testnet.tar.gz
tar -xvzf stabled-latest-linux-amd64-testnet.tar.gz
sudo mv stabled $HOME/go/bin
stabled version
```

### Initialize The Node

```bash
stabled init $MONIKER --chain-id stabletestnet_2201-1
```

### Download Genesis File

```bash
mv $HOME/.stabled/config/genesis.json $HOME/.stabled/config/genesis.json.backup

wget https://stable-testnet-data.s3.us-east-1.amazonaws.com/stable_testnet_genesis.zip
unzip stable_testnet_genesis.zip

cp genesis.json $HOME/.stabled/config/genesis.json
```

### Verify Genesis Checksum
Expected result `66afbb6e57e6faf019b3021de299125cddab61d433f28894db751252f5b8eaf2`

```bash
sha256sum $HOME/.stabled/config/genesis.json
```

### Download Configuration Files
Download optimized configuration.

```bash
wget https://stable-testnet-data.s3.us-east-1.amazonaws.com/rpc_node_config.zip unzip rpcnode_config.zip
```

Backup original config.

```bash
cp $HOME/.stabled/config/config.toml $HOME/.stabled/config/config.toml.backup
```

Copy the new configuration.

```bash
cp config.toml $HOME/.stabled/config/config.toml
```

### Update The Configuration File
`config.toml` file

```bash
PEERS="b74b5f566b37e6af6c3bd0aa8111ff50b256fcc1@194.163.170.176:26656,209a09611e4ae574c9ef425bbf5d11b15dbdab2c@212.90.121.89:26656,414c66671c13cdaef1a6decdc1efd47d50e2d495@116.202.212.179:26656,cd89dd1aa4afb38e2226d5cabe5b7db047255129@65.21.237.124:12656,128accd3e8ee379bfdf54560c21345451c7048c7@37.187.147.22:26656,6d3d5b13e2f203dbf718d8c201e2af97bbc52dbe@45.84.138.149:26656,08ff2704dffcbfce0038fbd499758f15ec78472f@51.159.226.76:26656,9b9897064ed6a27f3e44d3269ebe9bc06e1ba233@217.76.55.225:26656,c9621f90fbe0cb20d50d434aca007a2159090cd4@104.152.209.189:26656,3c353744a5260e4fb3a9c176e9fd095f93255149@162.250.188.149:13656"

sed -i -E 's|^moniker\s*=.*|moniker = "'"$MONIKER"'"|' $HOME/.stabled/config/config.toml

sed -i -E '/^\[p2p]/,/^\[/ s|^#?\s*max_num_inbound_peers\s*=.*|max_num_inbound_peers = 50|' $HOME/.stabled/config/config.toml
sed -i -E '/^\[p2p]/,/^\[/ s|^#?\s*max_num_outbound_peers\s*=.*|max_num_outbound_peers = 30|' $HOME/.stabled/config/config.toml
sed -i -E "/^\[p2p]/,/^\[/ s|^#?\s*persistent_peers\s*=.*|persistent_peers = \"$PEERS\"|" "$HOME/.stabled/config/config.toml"
sed -i -E '/^\[p2p]/,/^\[/ s|^#?\s*pex\s*=.*|pex = true|' $HOME/.stabled/config/config.toml

sed -i -E '/^\[rpc]/,/^\[/ s|^#?\s*laddr\s*=.*|laddr = "tcp://0.0.0.0:26657"|' $HOME/.stabled/config/config.toml
sed -i -E '/^\[rpc]/,/^\[/ s|^#?\s*max_open_connections\s*=.*|max_open_connections = 900|' $HOME/.stabled/config/config.toml
sed -i -E '/^\[rpc]/,/^\[/ s|^#?\s*cors_allowed_origins\s*=.*|cors_allowed_origins = ["*"]|' $HOME/.stabled/config/config.toml
```

Verify the changes.

```bash
grep -E '^\s*moniker\s*=' "$HOME/.stabled/config/config.toml"
echo
sed -n '/^\[p2p]/,/^\[/p' "$HOME/.stabled/config/config.toml" \
  | grep -E '^\[p2p]|^\s*max_num_inbound_peers\s*=|^\s*max_num_outbound_peers\s*=|^\s*persistent_peers\s*=|^\s*pex\s*='
echo
sed -n '/^\[rpc]/,/^\[/p' "$HOME/.stabled/config/config.toml" \
  | grep -E '^\[rpc]|^\s*laddr\s*=|^\s*max_open_connections\s*=|^\s*cors_allowed_origins\s*='
```

`app.toml` file

```bash
sed -i -E 's|^#?\s*inter-block-cache\s*=.*|inter-block-cache = false|' "$HOME/.stabled/config/app.toml"
sed -i -E '/^\[json-rpc]/,/^\[/ s|^#?\s*enable\s*=.*|enable = true|' $HOME/.stabled/config/app.toml
sed -i -E '/^\[json-rpc]/,/^\[/ s|^#?\s*address\s*=.*|address = "0.0.0.0:8545"|' $HOME/.stabled/config/app.toml
sed -i -E '/^\[json-rpc]/,/^\[/ s|^#?\s*ws-address\s*=.*|ws-address = "0.0.0.0:8546"|' $HOME/.stabled/config/app.toml
sed -i -E '/^\[json-rpc]/,/^\[/ s|^#?\s*allow-unprotected-txs\s*=.*|allow-unprotected-txs = true|' $HOME/.stabled/config/app.toml
```

Verify the changes

```bash
grep -E "^inter-block-cache" "$HOME/.stabled/config/app.toml"
echo
sed -n '/^\[json-rpc]/,/^\[/p' "$HOME/.stabled/config/app.toml" \
  | grep -E '^\[json-rpc]|^\s*enable\s*=|^\s*address\s*=|^\s*ws-address\s*=|^\s*allow-unprotected-txs\s*='
```

### Create Service File

```bash
sudo tee /etc/systemd/system/stabled.service > /dev/null <<EOF
[Unit]
Description=Stable Daemon Service
After=network-online.target

[Service]
User=$USER
ExecStart=/usr/bin/stabled start --chain-id stabletestnet_2201-1
Restart=always
RestartSec=3
LimitNOFILE=65535
StandardOutput=journal
StandardError=journal
SyslogIdentifier=stabled

[Install]
WantedBy=multi-user.target
EOF
```

## Applying Snpashot
### Backup and Remove Current Data

```bash
mkdir -p $HOME/stable-backup

cp $HOME/.stabled/data/priv_validator_state.json $HOME/stable-backup/priv_validator_state.json.backup

rm -rf $HOME/.stabled/data/*
```

### Download The Snapshot

```bash
mkdir -p $HOME/snapshot
cd $HOME/snapshot

wget -c --progress=bar:force:noscroll \
  -O snapshot.tar.lz4 \
  https://vault.astrostake.xyz/testnet/stable/stable-testnet_snapshot.tar.lz4
```

### Extract Snapshot

```bash
pv snapshot.tar.lz4 | tar -I lz4 -xf - -C $HOME/.stabled/
```

### Restore `priv_validator_state.json` File
```bash
cp $HOME/stable-backup/priv_validator_state.json.backup $HOME/.stabled/data/priv_validator_state.json
```

### Clean Up

```bash
rm snapshot.tar.lz4
```

### Start The Node
```bash
sudo systemctl daemon-reload
sudo systemctl enable stabled
sudo systemctl start stabled
```

### Check Status

```bash
sudo systemctl status stabled
```

### Check Logs

```bash
sudo journalctl -u stabled -fo cat
```

-----------------------------------------------------------------

# Command List
## Maintenance
Get sync information
```bash
stabled status 2>&1 | jq .sync_info
```

Get node peer
```bash
echo $(stabled tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.stabled/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

Get live peers
```bash
curl -sS http://localhost:26657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.stabled/config/config.toml
```

Reset chain data
```bash
stabled tendermint unsafe-reset-all --keep-addr-book --home $HOME/.stabled --keep-addr-book
```

Remove node
> [!CAUTION]
> Before removing the node, be aware that all chain data will be erased. **Ensure you've created a backup of your priv_validator_key.json!**

```bash
cd $HOME
sudo systemctl stop stabled
sudo systemctl disable stabled
sudo rm /etc/systemd/system/stabled.service
sudo systemctl daemon-reload
sudo rm -f $(which stabled)
sudo rm -rf $HOME/.stabled
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2025 BlockHub. All rights reserved.
</p>
