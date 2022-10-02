```bash
sudo systemctl stop Cardchain
cd $HOME && rm -rf Cardchain
git clone https://github.com/DecentralCardGame/Cardchain && cd Cardchain
git checkout 4e2c35d094c196733843ca0180c2305652dcaba7
make build
sudo mv build/Cardchain $(which Cardchain)
cp /usr/local/bin/Cardchain $HOME/.Cardchain/cosmovisor/genesis/bin/Cardchain
sudo systemctl restart Cardchain && journalctl -fu Cardchain -o cat
```
