# 🚀 Полный гайд по установке ноды Arkeo Mainnet
Подходит для чистого сервера Ubuntu 20.04/22.04, права root обязательны.

# 🔧 1. Установка системных пакетов
sudo apt update && sudo apt upgrade -y
sudo apt install curl git jq lz4 npm build-essential -y

# ⚙️ 2. Установка Go
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.23.0.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh
echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.profile
source $HOME/.bash_profile
go version

# 📦 3. Сборка бинарника Arkeo
cd ~
rm -rf arkeo
git clone https://github.com/arkeonetwork/arkeo
cd arkeo
git fetch --all
git checkout v1.0.13
make instal

# 🛠 4. Настройка systemd-сервиса
tee /etc/systemd/system/arkeod.service > /dev/null << EOF
[Unit]
Description=Arkeo Mainnet node service
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=$(which arkeod) start --home $HOME/.arkeo
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable arkeod

# 🌐 5. Настройка клиента
arkeod config set client chain-id arkeo-main-v1
arkeod config set client keyring-backend file
arkeod config set client node tcp://localhost:26657

# 🚀 6. Инициализация ноды
arkeod init <ВАШ_МОНИКЕР> --chain-id arkeo-main-v1

# 📥 7. Загрузка genesis и addrbook
curl -Ls https://snapshots3.stakevillage.net/arkeo-main-v1/genesis.json > $HOME/.arkeo/config/genesis.json
curl -Ls https://snapshots3.stakevillage.net/arkeo-main-v1/addrbook.json > $HOME/.arkeo/config/addrbook.json

# 🔗 8. Сиды и пиры
seeds="4d2c67a1d732679826b2f71c833e94b3718c2b50@seed2.arkeo.network:26656,416bd4379fa4fa3e76e59e4415396f727463142e@seed.arkeo.network:26656"
sed -i -e "s|^seeds *=.*|seeds = \"$seeds\"|" $HOME/.arkeo/config/config.toml

peers="e21ebcb0b2694e7b316f2f8de883300cffc93b32@peer1.arkeo.network:26656,..."
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.arkeo/config/config.toml

# ⛽ 9. Минимальная цена за газ
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.001uarkeo\"|" $HOME/.arkeo/config/app.toml

# 🧹 10. Pruning (оптимизация хранения)
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  $HOME/.arkeo/config/app.toml

  # 🛡 11. Кастомные порты (опционально)
  CUSTOM_PORT=158
  Меняем порты:
  sed -i -e "s%^proxy_app = .*%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%" ... $HOME/.arkeo/config/config.toml
  sed -i -e "s%^address = .*%address = \"tcp://localhost:${CUSTOM_PORT}17\"%" ... $HOME/.arkeo/config/app.toml
  
  arkeod config set client node tcp://localhost:${CUSTOM_PORT}57

  # ✅ 12. Запуск ноды
  sudo systemctl start arkeod
  sudo journalctl -u arkeod -f --no-hostname -o cat

  # 🏁 Готово! Нода Arkeo установлена и синхронизируется.

