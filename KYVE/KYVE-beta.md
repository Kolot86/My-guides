# KYVE
```bash
sudo apt update && sudo apt upgrade -y
```
```bash
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```
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
```bash
NODENAME=Du_not_copypast
```
```bash
KYVE_PORT=49
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export KYVE_CHAIN_ID=kyve-beta" >> $HOME/.bash_profile
echo "export KYVE_PORT=${KYVE_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```bash
cd $HOME
wget https://nc2.breithecker.de/s/BY4Lzj8TAQzgJZm/download/chain_linux_amd64.tar.gz
tar -xvzf chain_linux_amd64.tar.gz
chmod +x chaind && mv ./chaind /usr/local/bin/chaind
```
```bash
chaind config chain-id $KYVE_CHAIN_ID
chaind config node tcp://localhost:${KYVE_PORT}657
```
```bash
chaind init $NODENAME --chain-id $KYVE_CHAIN_ID
```
```bash
SEEDS=""
PEERS="410bf0cb2cdb9a6e159c14b9d01531b9ecb1edd4@3.70.26.46:26656"; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.kyve/config/config.toml
```
```bash
rm $HOME/.kyve/config/genesis.json
wget https://nc2.breithecker.de/s/z3bDsQk8D6snyWA/download/genesis-v0.7.0-beta.json
mv genesis-v0.7.0-beta.json ~/.kyve/config/genesis.json
```
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

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${KYVE_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${KYVE_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${KYVE_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${KYVE_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${KYVE_PORT}660\"%" $HOME/.kyve/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${KYVE_PORT}317\"%; s%^address = \":8080\"%address = \":${KYVE_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${KYVE_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${KYVE_PORT}091\"%" $HOME/.kyve/config/app.toml
```
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.kyved/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.kyve/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.kyve/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.kyve/config/app.toml
```
```bash
sed -i 's/minimum-gas-prices = "0tkyve"/minimum-gas-prices = "0.0001tkyve"/g' $HOME/.kyve/config/app.toml
```
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.kyve/config/config.toml
```
```bash
chaind tendermint unsafe-reset-all --home $HOME/.kyve
```
```bash
tee <<EOF > /dev/null /etc/systemd/system/kyved.service
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



```bash
sudo systemctl daemon-reload
sudo systemctl enable kyved
sudo systemctl restart kyved && sudo journalctl -u kyved -f -o cat
```

```bash
chaind keys add $WALLET
```
```bash
chaind keys add $WALLET --recover
```
```bash
chaind keys list
```
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
```bash
chaind query bank balances $KYVE_WALLET_ADDRESS
```
```bash
chaind tx staking create-validator \
  --amount 1000000000000000000tkyve \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(chaind tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $KYVE_CHAIN_ID
  ```
  ### Commands
  ```bash
  journalctl -fu kyved -o cat
  ```
  ```bash
  sudo systemctl restart kyved
  ```
  ```bash
  sudo systemctl stop kyved
  ```
  # Delite 
```bash
sudo systemctl stop kyved
sudo systemctl disable kyved
sudo rm /etc/systemd/system/chaind* -rf
sudo rm $(which kyve) -rf
sudo rm $HOME/.kyve* -rf
sudo rm $HOME/kyve -rf
sed -i '/KYVE_/d' ~/.bash_profile
```
