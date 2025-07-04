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
## 👠 2. Установка Go
```bash
### Удаляем старую версию Go (если была)
sudo rm -rf /usr/local/go
### Скачиваем и устанавливаем Go 1.23.0
curl -Ls https://go.dev/dl/go1.23.0.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
### Создаём файл с переменной окружения для системного уровня
echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh
### Добавляем переменную окружения в пользовательский профиль
echo 'export PATH=$PATH:/usr/local/go/bin' >> $HOME/.profile
echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.profile
### Применяем изменения текущей сессии
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:$HOME/go/bin
### Проверяем версию Go
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
MONIKER=<ВАШ_МОНИКЕР>
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

peers="e21ebcb0b2694e7b316f2f8de883300cffc93b32@peer1.arkeo.network:26656,..."
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
### config.toml ###
sed -i -E "s%proxy_app = \".*\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%" $HOME/.arkeo/config/config.toml
sed -i -E "s%laddr = \"tcp://127.0.0.1:[0-9]+\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%" $HOME/.arkeo/config/config.toml
sed -i -E "s%pprof_laddr = \".*\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%" $HOME/.arkeo/config/config.toml
sed -i -E "s%laddr = \"tcp://0.0.0.0:[0-9]+\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%" $HOME/.arkeo/config/config.toml
sed -i -E "s%prometheus_listen_addr = \".*\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.arkeo/config/config.toml
### app.toml ###
sed -i -E "s%address = \"tcp://localhost:[0-9]+\"%address = \"tcp://localhost:${CUSTOM_PORT}17\"%" $HOME/.arkeo/config/app.toml
sed -i -E "s%address = \":[0-9]+\"%address = \":${CUSTOM_PORT}80\"%" $HOME/.arkeo/config/app.toml
sed -i -E "s%address = \"localhost:[0-9]+\"%address = \"localhost:${CUSTOM_PORT}90\"%" $HOME/.arkeo/config/app.toml
sed -i -E "s%address = \"localhost:[0-9]+\"%address = \"localhost:${CUSTOM_PORT}91\"%" $HOME/.arkeo/config/app.toml
sed -i -E "s%address = \"0.0.0.0:[0-9]+\"%address = \"0.0.0.0:${CUSTOM_PORT}45\"%" $HOME/.arkeo/config/app.toml
sed -i -E "s%ws-address = \"0.0.0.0:[0-9]+\"%ws-address = \"0.0.0.0:${CUSTOM_PORT}46\"%" $HOME/.arkeo/config/app.toml
### Настройка CLI адреса узла ###
arkeod config set client node tcp://localhost:${CUSTOM_PORT}57
```
---
## 💫 12. Подробная проверка с выводом строк и портов:
```bash
echo "config.toml — порты и адреса:"
grep -E "proxy_app|laddr|pprof_laddr|prometheus_listen_addr" -n $HOME/.arkeo/config/config.toml
echo -e "\napp.toml — адреса и порты:"
grep -E "address|ws-address" -n $HOME/.arkeo/config/app.toml
```
---
## 🌸 13. Запуск ноды
```bash
sudo systemctl start arkeod
sudo journalctl -u arkeod -f --no-hostname -o cat
```
---
# 🎀 Готово! Ваша нода Arkeo установлена и синхронизируется.
> *Даже самые серьёзные блокчейны нуждаются в женском подходе — красиво, аккуратно и со вкусом!* 💖 
