# OKP4
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
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```
## Moniker, whrite something cool
```bash
NODENAME=Do_not_copypaste
```
## Make your custom ports. You can chose from 10 to 65
```bash
OKP4_PORT=23
```
## Save and import variables

```bash
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export OKP4_CHAIN_ID=okp4-nemeton" >> $HOME/.bash_profile
echo "export OKP4_PORT=${OKP4_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Binaries

```bash
cd $HOME && \
wget https://github.com/okp4/okp4d/releases/download/v2.2.0/okp4d-2.2.0-linux-amd64.tar.gz && \
tar -xvzf okp4d-2.2.0-linux-amd64.tar.gz && \
cd $HOME/target/release && \
mv okp4d-2.2.0-linux-amd64 /usr/local/bin/okp4d && \
chmod +x $(which okp4d)
```
```bash
cd $HOME
sudo rm -rf okp4d-2.2.0-linux-amd64.tar.gz
```bash


## Config app

```bash
okp4d config chain-id $OKP4_CHAIN_ID
okp4d config node tcp://localhost:${OKP4_PORT}657
```
## For your own risk 
```bash
okp4d config keyring-backend file
```

## Init app

```bash
okp4d init $NODENAME --chain-id $OKP4_CHAIN_ID
```
## Seeds and peers

```bash
SEEDS=""
PEERS="024a57c0bb6d868186b6f627773bf427ec441ab5@65.108.2.41:36656,7da790c663d678cb064ff4fba04556dcf18bda2c@65.109.70.23:17656,5fbddca54548bf125ee96bb388610fe1206f087f@51.159.66.123:26656,dc14197ed45e84ca3afb5428eb04ea3097894d69@88.99.143.105:26656,2bfd405e8f0f176428e2127f98b5ec53164ae1f0@142.132.149.118:26656,501ad80236a5ac0d37aafa934c6ec69554ce7205@89.149.218.20:26656,a49302f8999e5a953ebae431c4dde93479e17155@162.19.71.91:26656,769f74d3bb149216d0ab771d7767bd39585bc027@185.196.21.99:26656,fff0a8c202befd9459ff93783a0e7756da305fe3@38.242.150.63:16656"; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.okp4d/config/config.toml
```
## Genesis

```bash
wget -O $HOME/.okp4d/config/genesis.json "https://raw.githubusercontent.com/okp4/networks/main/chains/nemeton/genesis.json"
```
## Cosmovisor

```bash
cd $HOME
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v0.1.0 
```
```bash 
cd $HOME/go/bin
sudo chmod +x cosmovisor 
sudo mv cosmovisor /usr/local/bin/cosmovisor
```
```bash
mkdir -p ~/.okp4d/cosmovisor/genesis/bin/ && \
echo "{}" > ~/.okp4d/cosmovisor/genesis/upgrade-info.json
```
```bash
cp $(which okp4d) $HOME/.okp4d/cosmovisor/genesis/bin/okp4d
```
## Custom ports 

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${OKP4_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${OKP4_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OKP4_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${OKP4_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${OKP4_PORT}660\"%" $HOME/.okp4d/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${OKP4_PORT}317\"%; s%^address = \":8080\"%address = \":${OKP4_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${OKP4_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${OKP4_PORT}091\"%" $HOME/.okp4d/config/app.toml
```
## Pruning

```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.okp4d/config/app.toml
```
## Indexing 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.okp4d/config/config.toml
```

## Minimum gas price

```bash
sed -E -i 's/minimum-gas-prices = \".*\"/minimum-gas-prices = \"0.001uknow\"/' $HOME/.okp4d/config/app.toml
```

## Prometheus

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.okp4d/config/config.toml
```
## Reset chain data

```bash
okp4d tendermint unsafe-reset-all --home $HOME/.okp4d
```
# Service

```bash
sudo tee <<EOF > /dev/null /etc/systemd/system/okp4d.service
[Unit]
Description=Fire-Starter
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) start
Restart=on-failure
RestartSec=3
LimitNOFILE=infinity

Environment="DAEMON_HOME=$HOME/.okp4d"
Environment="DAEMON_NAME=okp4d"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable okp4d
sudo systemctl restart okp4d && sudo journalctl -u okp4d -f -o cat
```
## synchronization status
```bash
okp4d status 2>&1 | jq .SyncInfo
```
# Validator stuff, in process
## Create-restore wallet

```bash
okp4d keys add $WALLET
```
```bash
okp4d keys add $WALLET --recover
```
```bash
okp4d keys list
```
## Load variables into the system
```bash
OKP4_WALLET_ADDRESS=$(okp4d keys show $WALLET -a)
```
```bash
echo 'export OKP4_WALLET_ADDRESS='${OKP4_WALLET_ADDRESS} >> $HOME/.bash_profile
```
```bash
OKP4_VALOPER_ADDRESS=$(okp4d keys show $WALLET --bech val -a)
```
```bash
echo 'export OKP4_VALOPER_ADDRESS='${OKP4_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Check balance
```bash
okp4d query bank balances $OKP4_WALLET_ADDRESS
```
## Faucet
```bash

```
```bash

```

## Register Validator
```bash
okp4d tx staking create-validator \
  --amount 10000000uknow \
  --from $WALLET \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.3" \
  --commission-rate "0.08" \
  --min-self-delegation "1" \
  --pubkey  $(okp4d tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $OKP4_CHAIN_ID --gas=200000 --fees 500uknow
  ```
  ## Edit validator
  ```bash
  okp4d tx staking edit-validator --identity=F606B3E4C51A1634 --website="https://twitter.com/Kolot86692580" --details="Wof Wof" --chain-id=$OKP4_CHAIN_ID --from=$WALLET --gas=200000 --fees 500uknow
  ```
  
  
  ## Peer list
   ```bash
  curl -sS http://localhost:${OKP4_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
   ```
   ## Validator list
   ```bash
   okp4d q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
   ```
  
  ### Staking, Delegation and Rewards
Delegate stake
Change

```bash
okp4d tx staking delegate $OKP4_VALOPER_ADDRESS 1000000uknow --from=$WALLET --chain-id=$OKP4_CHAIN_ID --gas=auto --fees 250uknow -y
```

Redelegate stake from validator to another validator
```bash
okp4d tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 1000000uknow --from=$WALLET --chain-id=$OKP4_CHAIN_ID --gas=auto --fees 250uknow -y
```

Withdraw all rewards
```bash
okp4d tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$OKP4_CHAIN_ID --gas=auto --fees 250uknow -y
```

Withdraw rewards with commision
```bash
okp4d tx distribution withdraw-rewards $OKP4_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$OKP4_CHAIN_ID --gas=auto --fees 250uknow -y
```
  
  ## Voting
  ```bash
  okp4d tx gov vote 150 yes --from $WALLET --chain-id=$OKP4_CHAIN_ID --fees 250uknow -y
  ```
  ```bash
  11A821E7B0804204680F7418AC1DDC08D94ED32A53AC054710C4AF8E6162A703
  ```
  
  ### Commands
  ```bash
  sudo journalctl -fu okp4d -o cat
  ```
  ```bash
  sudo systemctl restart okp4d
  ```
  ```bash
  sudo systemctl stop okp4d
  ```
  Unjail
  ```bash
  okp4d tx slashing unjail --from $WALLET --chain-id $OKP4_CHAIN_ID --gas=200000 --fees 250uknow -y
  ```
  ```bash
  
  ```
  # Delete 
```bash
sudo systemctl stop okp4d
sudo systemctl disable okp4d
sudo rm /etc/systemd/system/OKP4* -rf
sudo rm $(which okp4d) -rf
sudo rm $HOME/.okp4d* -rf
sudo rm $HOME/OKP4 -rf
sed -i '/OKP4_/d' ~/.bash_profile
```
## ufw
```

```
```
sudo ufw disable
```


