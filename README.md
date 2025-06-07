### 1. Güncellemeler ve Temel Araçlar
```
sudo apt update -y && sudo apt upgrade -y
sudo apt install curl git jq lz4 build-essential -y
```

### ✅ 2. Go Kurulumu (Go 1.22+ önerilir)
```
cd $HOME
wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
```
### Go versiyon kontrolü:
```
go version
```
### 3. TacChain Kodlarını Çek ve Derle
```
cd $HOME
rm -rf tacchain
git clone https://github.com/TacBuild/tacchain.git
cd tacchain
git checkout v0.0.11
make build
```
### ✅ 4. Cosmovisor Kurulumu
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
mkdir -p $HOME/.tacchaind/cosmovisor/genesis/bin
cp build/tacchaind $HOME/.tacchaind/cosmovisor/genesis/bin/
mkdir -p $HOME/.tacchaind/cosmovisor/upgrades/v0.0.11/bin
cp build/tacchaind $HOME/.tacchaind/cosmovisor/upgrades/v0.0.11/bin/tacchaind
sudo ln -s $HOME/.tacchaind/cosmovisor/genesis $HOME/.tacchaind/cosmovisor/current -f
sudo ln -s $HOME/.tacchaind/cosmovisor/current/bin/tacchaind /usr/local/bin/tacchaind -f
```
### ✅ 5. Servis Dosyası Oluştur
```
sudo tee /etc/systemd/system/tacchaind.service > /dev/null <<EOF
[Unit]
Description=TacChain Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=always
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.tacchaind"
Environment="DAEMON_NAME=tacchaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=$HOME/.tacchaind/cosmovisor/current/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable tacchaind
```
### ✅ 6. Node Başlatma ve Port Ayarları
```
echo 'export TAC_PORT="59"' >> ~/.bash_profile
source ~/.bash_profile
```
```
tacchaind init <node-adiniz> --chain-id tacchain_2391-1
```
### ✅ 7. Genesis & Addrbook İndir
```
curl -Ls https://raw.githubusercontent.com/TacBuild/tacchain/refs/heads/main/networks/tacchain_2391-1/genesis.json > $HOME/.tacchaind/config/genesis.json
```
### ✅ 8. Seed & Peer Ayarları
```
SEEDS=""
PEERS="9c32b3b959a2427bd2aa064f8c9a8efebdad4c23@206.217.210.164:45130,04a2152eed9f73dc44779387a870ea6480c41fe7@206.217.210.164:45140,5aaaf8140262d7416ac53abe4e0bd13b0f582168@23.92.177.41:45110,ddb3e8b8f4d051e914686302dafc2a73adf9b0d2@23.92.177.41:45120"

sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.tacchaind/config/config.toml
```
### ✅ 9. Pruning ve Timeout Ayarları
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.tacchaind/config/app.toml

sudo sed -i 's/timeout_commit = "5s"/timeout_commit = "2s"/' $HOME/.tacchaind/config/config.toml
```
### ✅ 10. Port Ayarları
```
sed -i.bak -e "s%:1317%:${TAC_PORT}317%g;
s%:8080%:${TAC_PORT}080%g;
s%:9090%:${TAC_PORT}090%g;
s%:9091%:${TAC_PORT}091%g;
s%:8545%:${TAC_PORT}545%g;
s%:8546%:${TAC_PORT}546%g;
s%:6065%:${TAC_PORT}065%g" $HOME/.tacchaind/config/app.toml
```
```
sed -i.bak -e "s%:26658%:${TAC_PORT}658%g;
s%:26657%:${TAC_PORT}657%g;
s%:6060%:${TAC_PORT}060%g;
s%:26656%:${TAC_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${TAC_PORT}656\"%;
s%:26660%:${TAC_PORT}660%g" $HOME/.tacchaind/config/config.toml
```
### ✅ 11. Snapshot (Hızlı Senkronizasyon)
```
curl -o - -L https://snapshot.corenodehq.xyz/tac_testnet/tac_snap.tar.lz4  | lz4 -c -d - | tar -x -C $HOME/.tacchaind
```
### ✅ 12. Node Başlat
```
sudo systemctl start tacchaind && sudo journalctl -u tacchaind -f --no-hostname -o cat
```
### ✅ 13. Cüzdan Oluştur (veya import)
Cüzdan oluşturmak için:
```
tacchaind keys add cüzdanadınıyaz
```
Veya mevcut bir cüzdanı import etmek için:
```
tacchaind keys add cüzdanadınıyaz --recover
```
⚠️ mnemonic kelimeleri mutlaka yedekle!

### ✅ 14. Ethereum Adresini Öğren 
```
echo "0x$(tacchaind debug addr $(tacchaind keys show cüzdanadınıyaz -a) | grep hex | awk '{print $3}')"
```
Bu komut sana 0x... formatında bir adres verecek. Bu adres ile faucetten token isteyeceksin.

### Public key'i alma komutu 
```
tacchaind tendermint show-validator
```
Bu komut sana çıktıda şöyle bir şey verecek(Örnektir)

{"@type":"/cosmos.crypto.ed25519.PubKey","key":"oWg2ISpLF405Jcm2vXV+2v4fnjodh6aafuIdeoW+rUw="}

```
nano validatortx.json
```
### Açılan dosyaya aşağıdaki şablonu yapıtırın ve gerekli yerleri düzenleyin
```
{
  "pubkey": {
    "@type": "/cosmos.crypto.ed25519.PubKey",
    "key": "BURAYA_tacchaind_tendermint_show-validator_KULLANARAK_ALDIĞIN_KEY"
  },
  "amount": "1000000utac",
  "moniker": "senin-validator-adi",
  "identity": "keybase_id",
  "website": "https://seninsiten.com",
  "security": "email@domain.com",
  "details": "Kendi açıklaman",
  "commission-rate": "0.1",
  "commission-max-rate": "0.2",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
```
CTRL X Y enter ile  kaydedip çık
### Herşey hazır ve sync olduysak artık validatörümüzü oluşturalım
```
tacchain# tacchaind tx staking create-validator validatortx.json \
  --from cüzdanadı \
  --chain-id tacchain_2391-1 \
  --node http://localhost:59657 \
  --gas auto --gas-adjustment 1.4 --fees 9503625000000000utac -y
```

### Kendimize delege etmek için:
```
tacchaind tx staking delegate tacvaloperadresi 9000000000000000000utac \
  --from cüzdanadınız \
  --chain-id tacchain_2391-1 \
  --node http://localhost:59657 \
  --gas auto --gas-adjustment 1.4 --fees 6130825000000000utac -y
```





