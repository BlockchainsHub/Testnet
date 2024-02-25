# Cross Finance Validator Guide

## Hardware Requirements:
It is recommended to use ubuntu operating system.
* 32GB of Memory
* 500GB - 2TB SSD

## Dependencies
There are a few dependencies that need to be installed to smoothly run the validator node.

### Installing Git
```
sudo apt install git-all
```

### Installing Screen
```
sudo apt install screen
```

## Initialize Chain
* Download the node software.
```
wget https://github.com/crossfichain/crossfi-node/releases/download/v0.3.0-prebuild3/crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz && tar -xf crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz
```
* Clone the repository.
```
git clone https://github.com/crossfichain/testnet.git
```
* Create new screen to run the node.
```
screen -S crossfi
```
* Run the node.
```
./bin/crossfid start --home ./testnet
```
You can detached from the screen by pressing `ctrl + a + d `

## Import Your Account
If you have generated your wallet on https://test.xficonsole.com/. You can recover the account using the command below.
```
./bin/crossfid --home ./testnet keys add <wallet_name> --recover
```
Example:
```
./bin/crossfid --home ./testnet keys add kamuscrypto --recover
```
Enter your seed phrase.

![CleanShot 2024-02-25 at 18 17 36@2x](https://github.com/Kamus-Crypto/Testnet/assets/77204008/30539d5d-f41a-4097-afb5-ec22d5435bdd)
Enter your password.

![CleanShot 2024-02-25 at 18 22 25@2x](https://github.com/Kamus-Crypto/Testnet/assets/77204008/151fdb41-862d-4862-943f-82e2564e3ccf)

## Create Your Validator
After receiving your $MPX tokens from the Cross Finance team, you can continue to create your validator.

* Check your node sync status.
```
./bin/crossfid --home ./testnet status 2>&1 | jq .SyncInfo
```

`"Catching_up": false` means that you are synced with the network and can continue to create your validator. If it's `true` you have to wait till it changed to `false` before continue creating your validator.

![CleanShot 2024-02-25 at 18 31 12@2x](https://github.com/Kamus-Crypto/Testnet/assets/77204008/da7f26b8-6f2a-472b-8ebe-7bf100f8d617)


* Create your validator.
```
./bin/crossfid --home ./testnet tx staking create-validator \
  --amount=1000000000000000000mpx \
  --pubkey=$(./bin/crossfid --home ./testnet tendermint show-validator) \
  --moniker="your_moniker" \
  --chain-id="crossfi-evm-testnet-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas="auto" \
  --gas-prices="10000000000000mpx" \
  --gas-adjustment=1.5 \
  --from="your_wallet"
```
Example:
```
./bin/crossfid --home ./testnet tx staking create-validator \
  --amount=1000000000000000000mpx \
  --pubkey=$(./bin/crossfid --home ./testnet tendermint show-validator) \
  --moniker="kamuscrypto" \
  --chain-id="crossfi-evm-testnet-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas="auto" \
  --gas-prices="10000000000000mpx" \
  --gas-adjustment=1.5 \
  --from=kamuscrypto
```
