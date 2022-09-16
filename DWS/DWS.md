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
## Moniker, whrite some name
```bash
NODENAME=Kolot
```
## Make your custom ports. You can chose from 10 to 65
```bash
DWS_PORT=13
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
PEERS="b75df8fd5245235b5deb66f30afb395fec45e3fb@185.9.144.138:26656"; \
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
wget https://github.com/KYVENetwork/chain/releases/download/v0.0.1/cosmovisor_linux_amd64 && \
mv cosmovisor_linux_amd64 cosmovisor 
```
```bash
chmod +x cosmovisor && mv ./cosmovisor /usr/local/bin/cosmovisor
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
pruning_interval="50"
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

## Validator stuff, in process

```bash
dewebd keys add $WALLET
```
```bash
dewebd keys add $WALLET --recover
```
```bash
dewebd keys list
```
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
```bash
dewebd query bank balances $DWS_WALLET_ADDRESS
```
```bash
dewebd tx staking create-validator \
  --amount 1000000000tkyve \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(dewebd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $DWS_CHAIN_ID
  ```
  ## Edit validator
  ```bash
  dewebd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$DWS_CHAIN_ID \
  --from=$WALLET
  ```
  ## Peer list
   ```bash
  curl -sS http://localhost:${DWS_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
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
sudo rm /etc/systemd/system/dewebd* -rf
sudo rm $(which deweb) -rf
sudo rm $HOME/.deweb* -rf
sudo rm $HOME/deweb -rf
sed -i '/DWS_/d' ~/.bash_profile
```
