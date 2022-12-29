<p align="center">
  <img height="100" height="auto" src="https://github.com/Kolot86/My-guides/blob/main/Pictures/okp4.png">
</p>

## OKP4 Snapshot guide 

Here I going to explain how we can create a snapshot for OKP4 node.

Why a snapshot and not a state sync?

OKP4 has a CosmWasm smart contract implementation and if you start your node from state sync, then you have the risk that it’s going to crash. It can happen because of a request to some old contract in the wasm folder and if you start from the state sync your wasm folder is empty. You need to download wasm separately, or it’s better to use a snapshot – download the database with wasm folder.  


First, we need to have a fully synced OKP4 node, to install it you can use the *[official documentation]( https://docs.okp4.network/nodes/introduction)*.

After your node is installed and fully synced, you need to stop it:
```
sudo systemctl stop okp4d
```

Now we need to create a **screen session** where we going to serve our archive.
We can name this session OKP4:
```
screen -S okp4
```

Now let’s create an archive with one command:
```
cd $HOME && \
mkdir okp4-archive && \
cd okp4-archive && \
tar -zcvf okp4-archive.tar.gz $HOME/.okp4d/data $HOME/.okp4d/wasm && \
echo -e "\033[0;31m wget http://$(wget -qO- eth0.me):8000/okp4-archive.tar.gz \033[0m" && \
python3 -m http.server 8000
```

After its done, we can detach from the screen session by pressing **ctrl a + d**

Copy also **addrbook.json** which will be needed on the new server 

Yor archive will be available on this address: 

```
http://your_node_ip:8000/okp4-archive.tar.gz
```
---
### Start OKP4 from snapshot 

Now we can start our node on the new server from snapshot.

First, we stop the node: 

```
sudo systemctl stop okp4d
```
I suggest you make the same pruning adjustment on your node as on the node where you serve your snapshot. If you run without pruning - do nothing. 
In my case the pruning is tuned like this:
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.okp4d/config/app.toml
```

You can also copy the **addrbook.json** from the old server to a new one so your node can find peers after the restart.

We going to reset our node. This will erase your node database. If you are already running validator, be sure you backed up your **priv_validator_key.json** The command does not wipe the file. However, you should have a backup of it.

```
okp4d tendermint unsafe-reset-all --home $HOME/.okp4d --keep-addr-book
```
Now we can download the archive.
```
cd $HOME
wget http://your_node_ip:8000/okp4-archive.tar.gz
```
After the download is finished, we can unpack the archive directly to our database location  
```
cd $HOME
tar -zxvf okp4-archive.tar.gz --strip-components 1
```
Delete archive
```
cd $HOME
sudo rm -rf okp4-archive.tar.gz
```
Restart your node 
```
sudo systemctl restart okp4d && sudo journalctl -u okp4d -f -o cat
```
If everything works correctly, your node should start from the same height at which snapshot was taken. 
