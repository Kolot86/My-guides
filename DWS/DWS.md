# DWS
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
DWS_PORT=11
```
## Save and import variables

```bash
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export DWS_CHAIN_ID=deweb-testnet-sirius" >> $HOME/.bash_profile
echo "export DWS_PORT=${DWS_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Binaries

```bash
cd $HOME
git clone https://github.com/deweb-services/deweb.git
cd deweb
git checkout v0.3.1
make build
sudo cp build/dewebd /usr/local/bin/dewebd
```

## Config app

```bash
dewebd config chain-id $DWS_CHAIN_ID
dewebd config node tcp://localhost:${DWS_PORT}657
```
## For your own risk 
```bash
dewebd config keyring-backend file
```

## Init app

```bash
dewebd init $NODENAME --chain-id $DWS_CHAIN_ID
```
## Seeds and peers

```bash
SEEDS=""
PEERS="f68e3850968edd258aab866d7697dd1f99e6e9fb@75.119.138.95:26656,4206e15a077492ec2d392d4e9142847409b46285@149.102.143.147:26656,6888c6103d08344e8abfa0474be48e09120cab02@146.19.24.52:14656,0d25212f510f8868b639861de96ccd31fc1ea4dc@65.21.61.242:26656,63064d9fe6bdffe6a85154592ec36be48cd63b9e@116.202.236.115:21046,a3245ded96e3642ff3f1d80f75f60ba7a58f8877@135.181.249.13:14656,1fb96e8d9fd32589a0b10b23dd9fc520151d75a3@62.171.182.95:14656,1d97083fcd4a2be02f18adf425d09f13c00effae@209.126.83.57:14656,c16affc35507ce5c504906fed1c478595efb4675@86.48.5.144:26656,42558363e2e153b8ad9c618d2e5335d03ff09a60@167.86.95.179:26656"; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.deweb/config/config.toml
```
## Genesis

```bash
rm $HOME/.deweb/config/genesis.json
cd $HOME
curl -s https://raw.githubusercontent.com/deweb-services/deweb/main/genesis.json > ~/.deweb/config/genesis.json
```
## Cosmovisor

```bash
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v0.1.0 
```
```bash 
cd $HOME/go/bin
```
```bash
sudo chmod +x cosmovisor 
sudo mv cosmovisor /usr/local/bin/cosmovisor
```
```bash
mkdir -p ~/.deweb/cosmovisor/genesis/bin/ && \
echo "{}" > ~/.deweb/cosmovisor/genesis/upgrade-info.json
```
```bash
cp /usr/local/bin/dewebd $HOME/.deweb/cosmovisor/genesis/bin/dewebd
```
## Custom ports 

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${DWS_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${DWS_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${DWS_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${DWS_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${DWS_PORT}660\"%" $HOME/.deweb/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${DWS_PORT}317\"%; s%^address = \":8080\"%address = \":${DWS_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${DWS_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${DWS_PORT}091\"%" $HOME/.deweb/config/app.toml
```
## Pruning

```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.deweb/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.deweb/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.deweb/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.deweb/config/app.toml
```
## Minimum gas price

```bash
sed -E -i 's/minimum-gas-prices = \".*\"/minimum-gas-prices = \"0.001udws\"/' $HOME/.deweb/config/app.toml
```

## Prometheus

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.deweb/config/config.toml
```
## Reset chain data

```bash
dewebd tendermint unsafe-reset-all --home $HOME/.deweb
```
# Service

```bash
sudo tee <<EOF > /dev/null /etc/systemd/system/dewebd.service
[Unit]
Description=Fire-Starter
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) start
Restart=on-failure
RestartSec=3
LimitNOFILE=infinity

Environment="DAEMON_HOME=$HOME/.deweb"
Environment="DAEMON_NAME=dewebd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable dewebd
sudo systemctl restart dewebd && sudo journalctl -u dewebd -f -o cat
```
## synchronization status
```bash
dewebd status 2>&1 | jq .SyncInfo
```
# Validator stuff, in process
## Create-restore wallet

```bash
dewebd keys add $WALLET
```
```bash
dewebd keys add $WALLET --recover
```
```bash
dewebd keys list
```
## Load variables into the system
```bash
DWS_WALLET_ADDRESS=$(dewebd keys show $WALLET -a)
```
```bash
echo 'export DWS_WALLET_ADDRESS='${DWS_WALLET_ADDRESS} >> $HOME/.bash_profile
```
```bash
DWS_VALOPER_ADDRESS=$(dewebd keys show $WALLET --bech val -a)
```
```bash
echo 'export DWS_VALOPER_ADDRESS='${DWS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Check balance
```bash
dewebd query bank balances $DWS_WALLET_ADDRESS
```
## Register Validator
```bash
dewebd tx staking create-validator \
  --amount 1000000udws \
  --from $WALLET \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.3" \
  --commission-rate "0.08" \
  --min-self-delegation "1" \
  --pubkey  $(dewebd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $DWS_CHAIN_ID --fees 250udws
  ```
  ## Edit validator
  ```bash
  dewebd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$DWS_CHAIN_ID \
  --from=$WALLET --fees 250udws
  ```
  ## Domain
 ```bash
 dewebd tx domain register do-not-copypast --data="$BasicData" --from=$WALLET --chain-id=$DWS_CHAIN_ID --gas 2100000 --fees 2100udws --output json -b block
 ```
  
  ## Peer list
   ```bash
  curl -sS http://localhost:${DWS_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
   ```
   ## Validator list
   ```bash
   dewebd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
   ```
   ## Delagate to your validator
   ```bash
  dewebd tx staking delegate $DWS_VALOPER_ADDRESS 1000000udws --from=$WALLET --chain-id=$DWS_CHAIN_ID --fees 250udws -y
  ```
  ### Commands
  ```bash
  sudo journalctl -fu dewebd -o cat
  ```
  ```bash
  sudo systemctl restart dewebd
  ```
  ```bash
  sudo systemctl stop dewebd
  ```
  Unjail
  ```bash
  dewebd tx slashing unjail --from $WALLET --chain-id $DWS_CHAIN_ID --gas=auto 
  ```
  ```bash
  
  ```
  # Delete 
```bash
sudo systemctl stop dewebd
sudo systemctl disable dewebd
sudo rm /etc/systemd/system/deweb* -rf
sudo rm $(which dewebd) -rf
sudo rm $HOME/.deweb* -rf
sudo rm $HOME/deweb -rf
sed -i '/DWS_/d' ~/.bash_profile
```
