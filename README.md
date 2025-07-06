# ✨ Полный гайд по установке ноды Arkeo Mainnet от прекрасной девушки 🌸
Для установки и запуска ноды Arkeo в mainnet требуются следующие минимальные аппаратные и программные требования:
### ***аппаратные требования:***
Процессор: 4 ядра CPU (рекомендуется высокая тактовая частота для лучшей производительности).
Оперативная память: 8 ГБ RAM.
Хранилище: минимум 100 ГБ свободного пространства (рекомендуется SSD или NVMe). В будущем может потребоваться больше места для хранения данных блокчейна.
### ***программные требования:***
Ubuntu 20.04/22.04, права root обязательны.
### ***стабильное широкополосное интернет-соединение с минимальной скоростью загрузки/выгрузки 10 МБ/с.***
---
> *Пусть ваши ноды будут так же прекрасны, как моя улыбка 💄💻*
---
## 💅 1. Установка системных пакетов
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git jq lz4 npm build-essential -y
```
---
## 👑 2. Установка Go
```bash
### Удаление старой версии Go (если была)
sudo rm -rf /usr/local/go
### Скачивание и установка Go 1.23.0
curl -Ls https://go.dev/dl/go1.23.0.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
### Создание файла с переменной окружения для системного уровня
echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh
### Добавление переменной окружения в пользовательский профиль
echo 'export PATH=$PATH:/usr/local/go/bin' >> $HOME/.profile
echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.profile
### Применение изменения текущей сессии
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:$HOME/go/bin
### Проверка версии Go
go version
```
---
## 💎 3. Сборка бинарника Arkeo
```bash
cd ~
rm -rf arkeo
git clone https://github.com/arkeonetwork/arkeo
cd arkeo
git fetch --all
git checkout v1.0.13
make install
/root/go/bin/arkeod version
```
---
## 👜 4. Настройка systemd-сервиса
```bash
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
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable arkeod
```
---
## 🌐 5. Настройка клиента
```bash
arkeod config set client chain-id arkeo-main-v1
arkeod config set client keyring-backend file
arkeod config set client node tcp://localhost:26657
```
---
## 👑 6. Инициализация ноды
```bash
cd
MONIKER=VashMoniker
arkeod init $MONIKER --chain-id arkeo-main-v1
```
---
## 📥 7. Загрузка genesis и addrbook
```bash
curl -Ls -o $HOME/.arkeo/config/addrbook.json https://snapshots.polkachu.com/addrbook/arkeo/addrbook.json
curl -Ls -o $HOME/.arkeo/config/genesis.json https://snapshots.polkachu.com/genesis/arkeo/genesis.json
```
---
## 💫 8. Сиды и пиры
```bash
seeds="4d2c67a1d732679826b2f71c833e94b3718c2b50@seed2.arkeo.network:26656,416bd4379fa4fa3e76e59e4415396f727463142e@seed.arkeo.network:26656"
sed -i -e "s|^seeds *=.*|seeds = \"$seeds\"|" $HOME/.arkeo/config/config.toml
peers="e21ebcb0b2694e7b316f2f8de883300cffc93b32@peer1.arkeo.network:26656,b8653eecacbe3f413046beb0e8b53d8f520c925e@peer2.arkeo.network:26656,f3037b238720c022be888890b7d8ecb516ae2a05@peer3.arkeo.network:26656,2e0a5e51ae1eabf527eb54632feb6a90ae0704ba@204.16.245.181:26656,4b60b22753c88f3cd6ba42dae8170e1a22429e76@141.95.3.94:26656,56a47e4ad462edfba90d0409785bc56196fdf376@51.89.98.102:55926,57c53dc1149c8696c839fc5a230579327d650e4c@65.109.114.178:26656,637609e9fe4618fe1d5c7c3564dc9ce4678abf61@142.132.251.87:15856,beaf7267d852cfbaaaaccbc1f92e785e5e0f0420@18.218.164.255:26656,de8f228211e72e8bb206e4f0f5e6e703cb2505eb@95.217.36.103:26656,e077b7ffdfcd6ea6826a126d0003a98fe0218bf7@213.239.194.132:15856,e87f1d4cfa4b7c70defa93dffefc450e2a1c1dc4@44.240.61.167:26656,fc03a34ea37cee3d97391cee11bffc792560cd61@46.4.32.57:26656,a3998b8a50765975be2be59954db0f6de66f92e3@5.161.246.27:36657"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.arkeo/config/config.toml
```
---
## 💖 9. Минимальная цена за газ
```bash
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.001uarkeo\"|" $HOME/.arkeo/config/app.toml
```
---
## 💼 10. Pruning (оптимизация хранения)
```bash
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  $HOME/.arkeo/config/app.toml
```
---
## 💋 11. Кастомные порты (опционально)
Меняем порты:
```bash
export CUSTOM_PORT=163  # твой порт
### Обновление config.toml ###
sed -i -e "s%proxy_app = \".*\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%;
s%laddr = \"tcp://127.0.0.1:.*\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%;
s%pprof_laddr = \".*\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%;
s%laddr = \"tcp://0.0.0.0:.*\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%;
s%prometheus_listen_addr = \".*\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.arkeo/config/config.toml
### Обновление app.toml ###
sed -i -e "s%address = \"tcp://localhost:.*\"%address = \"tcp://localhost:${CUSTOM_PORT}17\"%;
s%address = \":.*80\"%address = \":${CUSTOM_PORT}80\"%;
s%address = \"localhost:.*90\"%address = \"localhost:${CUSTOM_PORT}90\"%;
s%address = \"localhost:.*91\"%address = \"localhost:${CUSTOM_PORT}91\"%;
s%address = \"0.0.0.0:.*45\"%address = \"0.0.0.0:${CUSTOM_PORT}45\"%;
s%ws-address = \"0.0.0.0:.*46\"%ws-address = \"0.0.0.0:${CUSTOM_PORT}46\"%" $HOME/.arkeo/config/app.toml
### Настройка CLI ###
arkeod config set client node tcp://localhost:${CUSTOM_PORT}57
```
---
## 💫 12. Подробная проверка с выводом строк и портов если хотите проверить:
```bash
echo "config.toml — порты и адреса:"
grep -E "proxy_app|laddr|pprof_laddr|prometheus_listen_addr" -n $HOME/.arkeo/config/config.toml
echo -e "\napp.toml — адреса и порты:"
grep -E "address|ws-address" -n $HOME/.arkeo/config/app.toml
```
---
## 💫 13. Для быстрого запуска нужно найти свежий снепшот и выполнить следующие действия:
```bash
### Переход в рабочую папку ###
cd $HOME/.arkeo
### Скачивание архива ###
wget -O <имя_файла_для_сохранения> <адрес_ссылки>
### Распаковка архива ###
lz4 -d файл.tar.lz4 | tar -xvf -
### Удаление архива ###
rm arkeo_snapshot.tar.lz4
### Если установка ноды повторная, то сначала становить systemd-сервис ###
sudo systemctl stop arkeod
### Убедиться что вы находитесь в рабочей папке /.arkeo ###
cd $HOME/.arkeo
### Затем удалить старую базу 
rm -rf data/*
### Выполнить скачивание, распаковку и удаление архива как это описано в командах выше ###
```
---
## 🌸 14. Запуск ноды
```bash
sudo systemctl start arkeod
sudo journalctl -u arkeod -f --no-hostname -o cat
```
---
# 🎀 Готово! Ваша нода Arkeo установлена и синхронизируется.
> *Даже самые серьёзные блокчейны нуждаются в женском подходе — красиво, функционально и со вкусом!* 💖 
