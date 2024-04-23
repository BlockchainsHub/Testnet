![Github Banner Cross](https://github.com/Kamus-Crypto/Testnet/assets/77204008/c5ad8d1d-c534-4418-bbeb-07e6510f95ba)

# Cross Finance Validator Guide

## Hardware Requirements:
It is recommended to use ubuntu operating system.
* 32GB of Memory
* 500GB - 2TB SSD

## Auto Installation
```
curl -o crossfi.sh https://gist.githubusercontent.com/botxx15/09e26c9bc3753a31b5c66fdb2f262298/raw/e4bb131bb743e408c90cd26f791b4a1434ceb8d5/crossfi.sh && bash crossfi.sh
```

## Import Your Account
If you have generated your wallet on https://test.xficonsole.com/. You can recover the account using the command below.
```
crossfid keys add <wallet_name> --recover
```
Example:
```
crossfid keys add kamuscrypto --recover
```
Enter your seed phrase.

![CleanShot 2024-02-25 at 18 17 36@2x](https://github.com/Kamus-Crypto/Testnet/assets/77204008/30539d5d-f41a-4097-afb5-ec22d5435bdd)
Enter your password.

![CleanShot 2024-02-25 at 18 22 25@2x](https://github.com/Kamus-Crypto/Testnet/assets/77204008/151fdb41-862d-4862-943f-82e2564e3ccf)

## Create Your Validator
After receiving your $MPX tokens from the Cross Finance team, you can continue to create your validator.

* Check your node sync status.
```
crossfid status 2>&1 | jq .SyncInfo
```

`"Catching_up": false` means that you are synced with the network and can continue to create your validator. If it's `true` you have to wait till it changed to `false` before continue creating your validator.

![CleanShot 2024-02-25 at 18 31 12@2x](https://github.com/Kamus-Crypto/Testnet/assets/77204008/da7f26b8-6f2a-472b-8ebe-7bf100f8d617)


* Create your validator.
```
crossfid tx staking create-validator \
  --amount=1000000000000000000mpx \
  --pubkey=$(crossfid tendermint show-validator) \
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
crossfid tx staking create-validator \
  --amount=1000000000000000000mpx \
  --pubkey=$(crossfid tendermint show-validator) \
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

-----------------------------------------------------------------

<p align="center">
  &copy; 2023 BlockHub. All rights reserved.
</p>