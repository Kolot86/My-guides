```bash
mkdir -p $HOME/.hermes/bin
```
```bash
wget https://github.com/informalsystems/ibc-rs/releases/download/v1.0.0-rc.1/hermes-v1.0.0-rc.1-x86_64-unknown-linux-gnu.tar.gz && \
tar -C $HOME/.hermes/bin/ -vxzf hermes-v1.0.0-rc.1-x86_64-unknown-linux-gnu.tar.gz && \
echo 'export PATH="$HOME/.hermes/bin:$PATH"' >> $HOME/.bash_profile && \
source $HOME/.bash_profile
```
```bash
hermes version
```

```bash
wget https://raw.githubusercontent.com/neutron-org/neutron-contracts/neutron_audit_oak_19_09_2022_fixes/validator_test_upload_contract.sh
```

```bash
chmod +x validator_test_upload_contract.sh
```

```bash
./validator_test_upload_contract.sh
```

```bash
./validator_test_upload_contract.sh $HOME/neutron_validators_test.wasm
```

```bash
sudo tee $HOME/.hermes/config.toml > /dev/null <<EOF
[global]

# Specify the verbosity for the relayer logging output. Default: 'info'
# Valid options are 'error', 'warn', 'info', 'debug', 'trace'.
log_level = 'debug'

# Specify the mode to be used by the relayer. [Required]
[mode]

# Specify the client mode.
[mode.clients]

# Whether or not to enable the client workers. [Required]
enabled = true

# Whether or not to enable periodic refresh of clients. [Default: true]
# Note: Even if this is disabled, clients will be refreshed automatically if
#      there is activity on a connection or channel they are involved with.
refresh = true

# Whether or not to enable misbehaviour detection for clients. [Default: false]
misbehaviour = true

# Specify the connections mode.
[mode.connections]

# Whether or not to enable the connection workers for handshake completion. [Required]
enabled = true

# Specify the channels mode.
[mode.channels]

# Whether or not to enable the channel workers for handshake completion. [Required]
enabled = true

# Specify the packets mode.
[mode.packets]

# Whether or not to enable the packet workers. [Required]
enabled = true

# Parametrize the periodic packet clearing feature.
# Interval (in number of blocks) at which pending packets
# should be eagerly cleared. A value of '0' will disable
# periodic packet clearing. [Default: 100]
clear_interval = 100

# Whether or not to clear packets on start. [Default: false]
clear_on_start = true

# Toggle the transaction confirmation mechanism.
# The tx confirmation mechanism periodically queries the `/tx_search` RPC
# endpoint to check that previously-submitted transactions
# (to any chain in this config file) have delivered successfully.
# Experimental feature. Affects telemetry if set to false.
# [Default: true]
tx_confirmation = true

# The REST section defines parameters for Hermes' built-in RESTful API.
# https://hermes.informal.systems/rest.html
[rest]

# Whether or not to enable the REST service. Default: false
enabled = true

# Specify the IPv4/6 host over which the built-in HTTP server will serve the RESTful
# API requests. Default: 127.0.0.1
host = '127.0.0.1'

# Specify the port over which the built-in HTTP server will serve the restful API
# requests. Default: 3000
port = 3000

# The telemetry section defines parameters for Hermes' built-in telemetry capabilities.
# https://hermes.informal.systems/telemetry.html
[telemetry]

# Whether or not to enable the telemetry service. Default: false
enabled = false

# Specify the IPv4/6 host over which the built-in HTTP server will serve the metrics
# gathered by the telemetry service. Default: 127.0.0.1
host = '127.0.0.1'

# Specify the port over which the built-in HTTP server will serve the metrics gathered
# by the telemetry service. Default: 3001
port = 3001

# neutron quark-1 testnet
[[chains]]
id = 'quark-1'
# TODO: Enter your node's public RPC ADDRESS (ip OR host)
rpc_addr = 'http://127.0.0.1:33657/'
# TODO: Enter your node's public GRPC ADDRESS (ip OR host)
grpc_addr = 'http://127.0.0.1:33090/'
# TODO: Enter your node's public websocket ADDRESS (usually equals to ws://[rpc_addr]/websocket)
websocket_addr = 'ws://http://127.0.0.1:33657/websocket'
rpc_timeout = '10s'
account_prefix = 'neutron'
key_name = 'neutron-ibc-relayer'
store_prefix = 'ibc'
default_gas = 5000000
max_gas = 15000000
gas_price = { price = 0.004, denom = 'untrn' }
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '20s'
max_block_time = '10s'
trusting_period = '14days'
trust_threshold = { numerator = '1', denominator = '3' }
address_type = { derivation = 'cosmos' }

# cosmos hub testnet
[[chains]]
id = 'theta-testnet-001'
rpc_addr = 'http://164.90.146.43:26657/'
grpc_addr = 'http://164.90.146.43:9090/'
websocket_addr = 'ws://164.90.146.43:26657/websocket'
rpc_timeout = '10s'
account_prefix = 'cosmos'
key_name = 'cosmoshub-ibc-relayer'
store_prefix = 'ibc'
default_gas = 100000
max_gas = 3000000
gas_price = { price = 0.0025, denom = 'uatom' }
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 180000
clock_drift = '5s'
max_block_time = '10s'
trusting_period = '32hours'
trust_threshold = { numerator = '1', denominator = '3' }
address_type = { derivation = 'cosmos' }

# juno: comment out cosmoshub chain and uncomment block below to connect with it
[[chains]]
id = 'uni-5'
rpc_addr = 'https://juno-testnet-rpc.polkachu.com/'
grpc_addr = 'http://juno-testnet-grpc.polkachu.com:12690'
websocket_addr = 'wss://juno-testnet-rpc.polkachu.com/websocket'
rpc_timeout = '10s'
account_prefix = 'juno'
key_name = 'juno-ibc-relayer'
store_prefix = 'ibc'
default_gas = 100000
max_gas = 3000000
gas_price = { price = 0.1, denom = 'ujunox' }
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 180000
clock_drift = '5s'
max_block_time = '10s'
trusting_period = '18days'
trust_threshold = { numerator = '1', denominator = '3' }
address_type = { derivation = 'cosmos' }

[chains.packet_filter]
policy = 'allow'
list = [
    # allow relaying only for channels created by a certain contract  
    # TODO: fill in your INTERCHAIN_TXS_CONTRACT_ADDRESS below AFTER you run the test script (not the updload contract script,
    # the test script) and restart the relayer.
    # will be something like `['icacontroller-neutron14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9s5c2epq*', '*'],`
    ['icacontroller-сюда адрес интерчеин контракта*', '*'],
]
EOF
```
```bash
export NEUTRON_CHAIN_MNEMONIC=""
export NEUTRON_CHAIN_ID="quark-1" 
export NEUTRON_KEY_NAME="neutron-ibc-relayer" 

hermes keys add --chain $NEUTRON_CHAIN_ID --mnemonic-file <(echo "$NEUTRON_CHAIN_MNEMONIC") --key-name $NEUTRON_KEY_NAME
```
```bash
export UNI_CHAIN_MNEMONIC=""
export UNI_CHAIN_ID="uni-5" 
export UNI_KEY_NAME="juno-ibc-relayer" 


hermes keys add --chain $UNI_CHAIN_ID --mnemonic-file <(echo "$UNI_CHAIN_MNEMONIC") --key-name $UNI_KEY_NAME
```
```bash
export THETA_CHAIN_MNEMONIC=""
export THETA_CHAIN_ID="theta-testnet-001" 
export THETA_KEY_NAME="cosmoshub-ibc-relayer" 

hermes keys add --chain $THETA_CHAIN_ID --mnemonic-file <(echo "$THETA_CHAIN_MNEMONIC") --key-name $THETA_KEY_NAME
```

```bash
hermes create connection --a-chain quark-1 --b-chain uni-5
```
# Save example
```bash
SUCCESS Connection { #Если есть success значит соединение создано
    delay_period: 0ns,
    a_side: ConnectionSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "quark-1",
                version: 1,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-59",
        ),
        connection_id: Some(
            ConnectionId(
                "connection-30",
            ),
        ),
    },
    b_side: ConnectionSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "theta-testnet-001",
                version: 0,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-1140",
        ),
        connection_id: Some(
            ConnectionId(
                "connection-1061",
            ),
        ),
    },
}


```
```bash
hermes create connection --a-chain quark-1 --b-chain theta-testnet-001
```

```bash
#тоже скорее всего не нужно, но оставил пока
sudo tee /etc/systemd/system/hermesd.service > /dev/null <<EOF
[Unit]
Description=hermes
After=network-online.target

[Service]
User=$USER
ExecStart=$(which hermes) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable hermesd
sudo systemctl restart hermesd && journalctl -u hermesd -f -o cat #запускать пока не нужно но оставлю 
```
```bash
sudo systemctl stop hermesd
```

```bash
wget https://raw.githubusercontent.com/neutron-org/neutron-contracts/neutron_audit_oak_19_09_2022_fixes/validator_test.sh
```

```bash
chmod +x validator_test.sh
```

```bash
./validator_test.sh #надо будет прописать конекшен ид
```


