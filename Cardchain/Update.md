```bash
sudo systemctl stop Cardchain
wget https://github.com/DecentralCardGame/Cardchain/releases/download/v0.8/Cardchain_v0.8_linux_amd64.tar.gz
tar -xvzf Cardchain_v0.8_linux_amd64.tar.gz
mv $HOME/Cardchaind /usr/local/bin/Cardchain
sudo systemctl restart Cardchain && journalctl -fu Cardchain -o cat
```
