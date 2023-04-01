# Gitshock



///Güncelliyelim
```
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip wget
```

///Go İndirelim
```
wget https://go.dev/dl/go1.18.2.linux-amd64.tar.gz
```
///Go Kuralım
```
sudo tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz
```

///Dosyaları Çekiyoruz
```
git clone https://github.com/gitshock-labs/gitshock-ibc-yui $HOME/gitshockd
```
///Kuruyoruz
```
cd $HOME/gitshockd
make install
```
///Versiyon Kontrolü Yapıyoruz
```
newared version
```
///Ağa Katılıyoruz
```
export CHAIN_ID=arnhemia-01
export MONIKER_NAME="node-adınızı-yazınız"
newared init "${MONIKER_NAME}" --chain-id ${CHAIN_ID}
```
///Servis Dosyasını Oluşturuyoruz.
```
sudo tee /etc/systemd/system/gitshockd.service > /dev/null << EOF
[Unit]
Description=Neware Chain Daemon
After=network.target

[Service]
Type=simple
User=neware
ExecStart=$(which gitshockd) start 
Restart=on-abort

[Install]
WantedBy=multi-user.target

[Service]
LimitNOFILE=65535 
```
///Genesis ve Adressbook indiriyoruz
```
wget https://raw.githubusercontent.com/gitshock-labs/testnet-list/master/testnet/arnhemia -O $HOME/.gitshockd/config/genesis.json
wget https://raw.githubusercontent.com/gitshock-labs/testnet-list/master/testnet/arnhemia -O $HOME/.gitshockd/config/addrbook.json 
```
////Port değiştimeniz gerekiyorsa/////
```
YENI_PORT=38
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${YENI_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${YENI_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${YENI_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${YENI_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${YENI_PORT}660\"%" $HOME/.gitshockd/config/config.toml

sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${YENI_PORT}317\"%; s%^address = \":8080\"%address = \":${YENI_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${YENI_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${YENI_PORT}091\"%" $HOME/.gitshockd/config/app.toml
```


///Başlatıyoruz
```
sudo systemctl daemon-reload
sudo systemctl enable gitshockd
sudo systemctl start gitshockd
sudo journalctl -u gitshockd -f --no-hostname -o cat
```
///Cüzdan oluşturuyoruz
//Eğer Varsa ( --recover) edebilirisiniz
```
newared keys add cüzdanadı
```

///Validator Oluşturuyoruz
```
export PUBKEY=$( newared tendermint show-validator)
export CHAIN_ID=arnhemia-01
export MONIKER_NAME="node-adınızı-yazınız"
```
```
gitshockd tx staking create-validator 
        --moniker="${MONIKER_NAME}" \
        --amount=1000000ugtfx \
        --gas-prices=1ugtfx \
        --pubkey=$PUBKEY \
        --from=validator \
        --yes \
        --node=tcp://localhost:${YENI_PORT}657 \
        --chain-id=${CHAIN_ID} \
        --commission-max-change-rate=0.01 \
        --commission-max-rate=0.20 \
        --commission-rate=0.10 \
        --min-self-delegation=1 
```

