# Как поставить игру на бесплатный сервер (круглосуточно)

Игре нужен настоящий сервер с постоянным диском (там лежит база SQLite) и публичным HTTPS-адресом.
Подходят не все бесплатные хостинги, см. конец файла.

## Вариант 1. Oracle Cloud Always Free (рекомендую)
Бесплатная виртуальная машина без ограничения по времени. Нужна банковская карта для проверки личности
(кредитная или дебетовая, не виртуальная и не предоплаченная; деньги не списываются, пока не выйдешь за лимиты).
Иногда в выбранном регионе нет свободных машин, тогда нужно подождать и попробовать снова.

1. Зарегистрируйся на oracle.com/cloud/free, выбери домашний регион (потом не меняется).
2. Compute → Instances → Create instance. Образ Ubuntu 22.04/24.04. Форма: бесплатная
   (VM.Standard.E2.1.Micro с пометкой Always Free либо Ampere A1). Скачай SSH-ключ.
3. В сети (VCN → Security List) открой входящие порты TCP 80 и 443.
4. Подключись по SSH и поставь всё нужное:
   ```
   sudo apt update && sudo apt install -y curl unzip ufw
   curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash - && sudo apt install -y nodejs build-essential
   sudo npm i -g pm2
   sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT && sudo iptables -I INPUT -p tcp --dport 443 -j ACCEPT
   ```
5. Загрузи папку HotelTycoon на сервер (scp или WinSCP), затем:
   ```
   cd HotelTycoon && npm install
   cp .env.example .env && nano .env      # BOT_TOKEN, WEBAPP_URL=https://твой-домен, ADMIN_IDS
   pm2 start src/server.js --name hotel && pm2 save && pm2 startup
   ```
6. HTTPS. Нужно доменное имя, указывающее на IP сервера (можно бесплатный поддомен, например на DuckDNS).
   Самый простой способ, Caddy сам получает сертификат:
   ```
   sudo apt install -y caddy
   echo 'твой-домен.duckdns.org { reverse_proxy localhost:3000 }' | sudo tee /etc/caddy/Caddyfile
   sudo systemctl restart caddy
   ```
7. В BotFather проверь, что кнопка меню или Mini App указывает на `https://твой-домен...`.

## Вариант 2. Google Cloud (e2-micro Always Free)
Одна маленькая машина бесплатно, но только в некоторых регионах США, тоже нужна карта. Установка такая же, как выше.

## Вариант 3. Без карты: свой компьютер или старый телефон
Игра работает, пока устройство включено и в сети. Публичный адрес даёт бесплатный Cloudflare Tunnel
(постоянный адрес требует свой домен; временный меняется при каждом перезапуске, и тогда нужно менять WEBAPP_URL).

## Что не подходит
- Render Free: засыпает через 15 минут без запросов и стирает файлы (вместе с базой SQLite) при перезапуске.
  Рейды при этом не запускались бы.
- Бесплатные «ботхостинги» вроде FPS.ms: нужно вручную продлевать раз в 24 часа и нет публичного адреса для мини-аппа.

## Не потеряй базу
Игроки лежат в `data/hotel-tycoon.sqlite`. Раз в день копируй папку `data/` куда-нибудь ещё (cron + scp, или хотя бы руками).
