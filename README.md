# Nibiru-Testnet-2


Explorer:
>-  https://explorer.secardnode.com/nibiru


## Sistem Gereksinimleri
### Minimum Sistem Gereksinimler
 - 4x CPUs
 - 16GB RAM
 - 500GB of disk space (SSD)


### Manuel Kurulum

// İlk Öncelikle Gerekli Updateleri Yüklüyoruz.

~~~bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make gcc -y
~~~


// Go Kurulumu Yapıyoruz

~~~bash
cd $HOME
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm -rf  "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
~~~

// Binnary Dosyasını İndirip Build Ediyoruz

~~~bash
cd $HOME
git clone https://github.com/NibiruChain/nibiru
cd nibiru
git checkout v0.16.2
make install 
~~~

/// Yükledikten Sonra v0.16.2 olduğunu aşağıdaki kodu yazarak onaylayalım
~~~
nibid version
~~~

Genesisi İndiriyoruz

~~~bash
NETWORK=nibiru-testnet-2
curl -s https://networks.testnet.nibiru.fi/$NETWORK/genesis > $HOME/.nibid/config/genesis.json
~~~

Seed ve Peerleri Giriyoruz

~~~bash
NETWORK=nibiru-testnet-2
sed -i 's|seeds =.*|seeds = "'$(curl -s https://networks.testnet.nibiru.fi/$NETWORK/seeds)'"|g' $HOME/.nibid/config/config.toml
~~~



Pruning Ayarlarını Yapıyoruz

~~~bash
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.nibid/config/app.toml
~~~

Gerekli Chain Ayarlarını Yapıyoruz

~~~bash
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025unibi"/g' $HOME/.nibid/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.nibid/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.nibid/config/config.toml
~~~

Eski Data Varsa Siliyoruz

~~~bash
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
~~~

Ve Servis Dosyası Oluşturuyoruz

~~~bash
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=nibiru
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start --home $HOME/.nibid
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
~~~

Servis Dosyasını Aktif Edip Çalıştırıyoruz

~~~bash
sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl restart nibid && sudo journalctl -u nibid -f
~~~

## Wallet Oluşturma
Mnemonic Kelimelerinizi Güvenli Bir Yere Kaydedin Kaybetmeyin

~~~bash
nibid keys add walletisminiyazıyorsunuz
~~~

Eskiden Wallet Oluşturduysanız Recovery Edebilirsiniz

~~~bash
nibid keys add walletisminiyazıyorsunuz --recover
~~~

Sync Olduktan Sonra Discord Faucet Sayfanızdan Cüzdan Adresinize Faucet Talep Edip Validator Oluşturabilirsiniz


Validator Oluşturma

~~~bash
nibid tx staking create-validator \
  --amount 1000000unibi \
  --from cüzdanisminiziyazın \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(nibid tendermint show-validator) \
  --moniker SizinMonikerİsminizYazın \
  --chain-id nibiru-testnet-2 \
  --fees 10000unibi
~~~
  


### Servis Komutları
Log Kontrol

~~~bash
sudo journalctl -u nibid -f
~~~

Servis Durdurma

~~~bash
sudo systemctl stop nibid
~~~

Servis Başlatma

~~~bash
sudo systemctl start nibid
~~~

Servis Restart

~~~bash
sudo systemctl restart nibid
~~~

### Cüzdan

Cüzdan Miktar Kontrol

~~~bash
nibid query bank balances cüzdanadresiniz
~~~


### Node Silme

~~~bash
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm -rf /etc/systemd/system/nibid*
sudo rm $(which nibid)
sudo rm -rf $HOME/.nibid
sudo rm -fr $HOME/nibiru
sed -i "/NIBIRU_/d" $HOME/.bash_profile
~~~
