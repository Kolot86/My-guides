# LOYAL
## Packages
```bash
sudo apt update && sudo apt upgrade -y
```
## Dependencies
```bash
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```
## Go

```bash
ver="1.19.1" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
## Moniker, whrite something cool
```bash
NODENAME=Do_not_copypaste
```
## Save and import variables

```bash
LOYAL_PORT=13
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export LOYAL_CHAIN_ID=loyal-1" >> $HOME/.bash_profile
echo "export LOYAL_PORT=${LOYAL_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Binaries
```bash
cd $HOME && \
wget https://github.com/LoyalLabs/loyal/releases/download/v0.25.1/loyal_v0.25.1_linux_amd64.tar.gz && \
tar -xvzf loyal_v0.25.1_linux_amd64.tar.gz && \
sudo mv loyald /usr/local/bin/loyald && \
chmod +x $(which loyald)
```
```bash
cd $HOME
sudo rm -rf loyal_v0.25.1_linux_amd64.tar.gz
```
## Config app

```bash
loyald config chain-id $LOYAL_CHAIN_ID
loyald config node tcp://localhost:${LOYAL_PORT}657
```
## For your own risk 
```bash
loyald config keyring-bac
## Seeds and peers

```bash
SEEDS=""
PEERS="7490c272d1c9db40b7b9b61b0df3bb4365cb63a6@loyal-seed.netdots.net:26656,b66ecdf36bb19a9af0460b3ae0901aece93ae006@pubnode1.joinloyal.io:26656"; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.loyal/config/config.toml
```
## Genesis

```bash
wget -O $HOME/.loyal/config/genesis.json "https://raw.githubusercontent.com/LoyalLabs/net/main/mainnet/genesis.json"
```
## Custom ports 

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${LOYAL_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${LOYAL_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${LOYAL_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${LOYAL_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${LOYAL_PORT}660\"%" $HOME/.loyal/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${LOYAL_PORT}317\"%; s%^address = \":8080\"%address = \":${LOYAL_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${LOYAL_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${LOYAL_PORT}091\"%" $HOME/.loyal/config/app.toml
```
## Pruning

```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.loyal/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.loyal/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.loyal/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.loyal/config/app.toml
```
## Indexing 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.loyal/config/config.toml
```

## Minimum gas price

```bash
sed -E -i 's/minimum-gas-prices = \".*\"/minimum-gas-prices = \"0.001ulyl\"/' $HOME/.loyal/config/app.toml
```

## Prometheus

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.loyal/config/config.toml
```
## Reset chain data

```bash
loyald tendermint unsafe-reset-all --home $HOME/.loyal
```
```bash
sudo tee /etc/systemd/system/loyald.service > /dev/null <<EOF
[Unit]
Description=Stake to Kolot
After=network-online.target

[Service]
User=$USER
ExecStart=$(which loyald) start --home $HOME/.loyal
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
## Register and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable loyald
sudo systemctl restart loyald && sudo journalctl -u loyald -f -o cat
```
## synchronization status
```bash
loyald status 2>&1 | jq .SyncInfo
```
# Validator stuff, in process
## Create-restore wallet

```bash
loyald keys add $WALLET
```
```bash
loyald keys add $WALLET --recover
```
```bash
loyald keys list
```
## Load variables into the system
```bash
LOYAL_WALLET_ADDRESS=$(loyald keys show $WALLET -a)
```
```bash
echo 'export LOYAL_WALLET_ADDRESS='${LOYAL_WALLET_ADDRESS} >> $HOME/.bash_profile
```
```bash
LOYAL_VALOPER_ADDRESS=$(loyald keys show $WALLET --bech val -a)
```
```bash
echo 'export LOYAL_VALOPER_ADDRESS='${LOYAL_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Check balance
```bash
loyald query bank balances $LOYAL_WALLET_ADDRESS
```
## Register Validator
```bash
loyald tx staking create-validator \
  --amount 1000000ulyl \
  --from $WALLET \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.3" \
  --commission-rate "0.08" \
  --min-self-delegation "1" \
  --pubkey  $(loyald tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $LOYAL_CHAIN_ID --gas=200000 --fees 500ulyl
  ```
## Peer list
   ```bash
  curl -sS http://localhost:${LOYAL_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
   ```
   ## Validator list
   ```bash
   loyald q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
   ```
  
  ## Staking, Delegation and Rewards
Delegate stake
```bash
loyald tx staking delegate $LOYAL_VALOPER_ADDRESS 1000000ulyl --from=$WALLET --chain-id=$LOYAL_CHAIN_ID --gas=200000 --fees 500ulyl -y
```

Redelegate stake from validator to another validator
```bash
loyald tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 1000000ulyl --from=$WALLET --chain-id=$LOYAL_CHAIN_ID --gas=200000 --fees 250ulyl -y
```

### Withdraw rewards with commision
```bash
loyald tx distribution withdraw-rewards $LOYAL_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$LOYAL_CHAIN_ID --gas=200000 --fees 500ulyl -y
```
```bash
loyald tx bank send $WALLET <wallet> 1000000ulyl --gas=200000 --fees 500ulyl -y
```
  
  ## Voting
  ```bash
  loyald tx gov vote 1 yes --from $WALLET --chain-id=$LOYAL_CHAIN_ID --fees 250ulyl -y
### Commands
  ```bash
  sudo journalctl -fu loyald -o cat
  ```
  ```bash
  sudo systemctl restart loyald
  ```
  ```bash
  sudo systemctl stop loyald
  ```
  Unjail
  ```bash
  loyald tx slashing unjail --from $WALLET --chain-id $LOYAL_CHAIN_ID --gas=200000 --fees 250ulyl -y
  ```
  ```bash
  
  ```
  # Delete 
```bash
sudo systemctl stop loyald
sudo systemctl disable loyald
sudo rm /etc/systemd/system/loyald* -rf
sudo rm $(which loyald) -rf
sudo rm $HOME/.loyal* -rf
sudo rm $HOME/LOYAL -rf
sed -i '/LOYAL_/d' ~/.bash_profile
```
