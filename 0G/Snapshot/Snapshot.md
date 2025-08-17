![0G Github Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/34a32724-b411-41e4-8696-e390dfa01cab)

# 0G Snapshot Guide
This guide walks you through installing a snapshot for the 0G testnet blockchain network. Snapshots provide a compressed backup of blockchain data, allowing you to quickly sync your node without downloading the entire blockchain history from genesis.

> [!NOTE]
> 0G testnet snapshots are updated daily at 02:00 UTC, ensuring you always have access to recent blockchain state with minimal catch-up required.

### Step 1: Install Dependencies
Install all the dependencies before begin the snapshot installation.
```bash
sudo apt update && sudo apt upgrade
sudo apt install pv lz4 aria2c
```
### Step 2: Check Current Snapshot Information
Before downloading, check the latest snapshot details:

```bash
curl -s https://0g-testnet-snapshot.blockhub.id/snapshot/info | jq
```

### Step 3: Stop Existing Services
Stop both services

```bash
sudo systemctl stop 0gchaind.service
sudo systemctl stop geth.service
```

Verify they're stopped

```bash
sudo systemctl status 0gchaind.service
sudo systemctl status geth.service
```

### Step 4: Backup `priv_validator_state.json`

```bash
mkdir -p $HOME/backup

cp $HOME/.0gchaind/galileo/0g-home/0gchaind-home/data/priv_validator_state.json $HOME/backup/priv_validator_state.json.backup
```

### Step 5: Download the Snapshot
Create a temporary directory for download

```bash
mkdir -p ~/0g-snapshot-temp
cd ~/0g-snapshot-temp
```

Download with resume capability

```bash
aria2c -x 16 -s 16 -k 1M --continue=true https://0g-testnet-snapshot.blockhub.id/snapshot/latest
```

### Step 6: Extract the Snapshot
Extract the downloaded archive

```bash
lz4 -dc latest | pv | tar -xf -
```

### Step 7: Install Snapshot Data
Copy the extracted data to your node directories. **Keep in mind that your node directories may different from this setup depends on your initial setup**.

```bash
cp -r ~/0g-snapshot-temp/0gchaind-data/* $HOME/.0gchaind/galileo/0g-home/0gchaind-home/data/

cp -r ~/0g-snapshot-temp/geth-data/* $HOME/.0gchaind/galileo/0g-home/geth-home/geth/
```

### Step 8: Start Services
Start the services in the correct order

```bash
sudo systemctl start geth.service

sleep 15

sudo systemctl start 0gchaind.service
```

### Step 9: Check Logs
Check Geth service logs

```bash
sudo journalctl -u geth -fo cat
```

Check 0gchaind service logs

```bash
sudo journalctl -u 0gchaind -fo cat
```

### Step 10: Cleanup Snapshot
Remove the temporary directory

```bash
cd $HOME
rm -rf ~/0g-snapshot-temp
```
