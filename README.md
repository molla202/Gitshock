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
///Kullanıcı Yapılandırma
```
sudo useradd -m neware
sudo su -s /bin/bash -l neware
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:$(go env GOPATH)/bin
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
        --node=tcp://localhost:26657 \
        --chain-id=${CHAIN_ID} \
        --commission-max-change-rate=0.01 \
        --commission-max-rate=0.20 \
        --commission-rate=0.10 \
        --min-self-delegation=1 
```

