# KYVE
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
NODENAME=Du_not_copypast
```
## Make your custom ports. You can chose from 10 to 65
```bash
KYVE_PORT=26
```
## Save and import variables

```bash
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export KYVE_CHAIN_ID=kyve-beta" >> $HOME/.bash_profile
echo "export KYVE_PORT=${KYVE_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Binaries

```bash
cd $HOME
wget https://nc2.breithecker.de/s/BY4Lzj8TAQzgJZm/download/chain_linux_amd64.tar.gz
tar -xvzf chain_linux_amd64.tar.gz
sudo chmod +x chaind && sudo mv ./chaind /usr/local/bin/chaind
```
## Config app

```bash
chaind config chain-id $KYVE_CHAIN_ID
chaind config node tcp://localhost:${KYVE_PORT}657
```
## For your own risk 
```bash
chaind config keyring-backend test
```

## Init app

```bash
chaind init $NODENAME --chain-id $KYVE_CHAIN_ID
```
## Seeds and peers

```bash
SEEDS=""
PEERS="9e01357936c3db51f9a2b040497599d56d83090c@136.243.103.53:24656,3e97e4489e68335c5a69fbdac0d67502079e18ae@5.161.47.249:26656,897621af43a1ab19a9e8439f0d0d725cf6f558ab@80.82.215.243:26656,68c97012e28600c22251ffd298afb17f866bfb29@168.119.115.250:21656,a89582698a6b46b2c353a721f48f8af9ebb8fbe8@135.181.89.153:26656,09aae3c348b1fbd6153dc141b733a1e6e3d21b06@5.161.134.175:26656,4d7740c5ba34ade97bb6491422eab414b7831ca0@135.181.5.47:20056"; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.kyve/config/config.toml
```
## Genesis

```bash
sudo rm $HOME/.kyve/config/genesis.json
wget https://nc2.breithecker.de/s/z3bDsQk8D6snyWA/download/genesis-v0.7.0-beta.json
mv genesis-v0.7.0-beta.json ~/.kyve/config/genesis.json
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
mkdir -p ~/.kyve/cosmovisor/genesis/bin/ && \
echo "{}" > ~/.kyve/cosmovisor/genesis/upgrade-info.json
```
```bash
cp /usr/local/bin/chaind $HOME/.kyve/cosmovisor/genesis/bin/chaind
```
## Custom ports 

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${KYVE_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${KYVE_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${KYVE_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${KYVE_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${KYVE_PORT}660\"%" $HOME/.kyve/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${KYVE_PORT}317\"%; s%^address = \":8080\"%address = \":${KYVE_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${KYVE_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${KYVE_PORT}091\"%" $HOME/.kyve/config/app.toml
```
## Pruning

```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.kyve/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.kyve/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.kyve/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.kyve/config/app.toml
```
## Minimum gas price

```bash
sed -i 's/minimum-gas-prices = "0tkyve"/minimum-gas-prices = "0.0001tkyve"/g' $HOME/.kyve/config/app.toml
```

## Prometheus

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.kyve/config/config.toml
```
## Reset chain data

```bash
chaind tendermint unsafe-reset-all --home $HOME/.kyve
```
# Service

```bash
sudo tee <<EOF > /dev/null /etc/systemd/system/kyved.service
[Unit]
Description=Fire-Starter
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) start
Restart=on-failure
RestartSec=3
LimitNOFILE=infinity

Environment="DAEMON_HOME=$HOME/.kyve"
Environment="DAEMON_NAME=chaind"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable kyved
sudo systemctl restart kyved && sudo journalctl -u kyved -f -o cat
```

## synchronization status
```bash
chaind status 2>&1 | jq .SyncInfo
```

# Validator stuff, in process
## Create-restore wallet

```bash
chaind keys add $WALLET
```
```bash
chaind keys add $WALLET --recover
```
```bash
chaind keys list
```
## Load variables into the system
```bash
KYVE_WALLET_ADDRESS=$(chaind keys show $WALLET -a)
```
```bash
echo 'export KYVE_WALLET_ADDRESS='${KYVE_WALLET_ADDRESS} >> $HOME/.bash_profile
```
```bash
KYVE_VALOPER_ADDRESS=$(chaind keys show $WALLET --bech val -a)
```
```bash
echo 'export KYVE_VALOPER_ADDRESS='${KYVE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Check balance
```bash
chaind query bank balances $KYVE_WALLET_ADDRESS
```
## Register validator
```bash
chaind tx staking create-validator \
  --amount 1000000000tkyve \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(chaind tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $KYVE_CHAIN_ID
  ```
  ## Edit validator
  ```bash
  chaind tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$KYVE_CHAIN_ID \
  --from=$WALLET
  ```
  ## Rewards, Staking, Delegation
  ```bash
  chaind tx distribution withdraw-rewards $KYVE_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$KYVE_CHAIN_ID --gas=200000 --fees 200000tkyve -y
  ```
  ```bash
  chaind tx staking delegate $KYVE_VALOPER_ADDRESS 1000000000000tkyve --from=$WALLET --chain-id=$KYVE_CHAIN_ID --gas=200000 --fees 200000tkyve -y
  ```
  ## Get list of validators
```
chaind q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
## Governance
#### Chech proposals 
```
chaind q gov proposals
```
#### Voite
```
chaind tx gov vote 27 yes --from=$WALLET --gas=200000 --fees 200000tkyve -y
```

  ## Peer list
   ```bash
  curl -sS http://localhost:${KYVE_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
   ```
  ### Commands
  ```bash
  sudo journalctl -fu kyved -o cat
  ```
  ```bash
  sudo systemctl restart kyved
  ```
  ```bash
  sudo systemctl stop kyved
  ```
  Unjail
  ```bash
  chaind tx slashing unjail --from $WALLET --chain-id $KYVE_CHAIN_ID --gas=200000 --fees 200000tkyve -y
  ```
  ```bash
  
  ```
  # Delete 
```bash
sudo systemctl stop kyved
sudo systemctl disable kyved
sudo rm /etc/systemd/system/chaind* -rf
sudo rm $(which kyved) -rf
sudo rm $HOME/.kyve* -rf
sudo rm $HOME/kyve -rf
sed -i '/KYVE_/d' ~/.bash_profile
```
