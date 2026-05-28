# Telegram-Bot-VPN
# 🔒 LockBox VPN Bot

Telegram-бот для управления приватным VPN-сервисом. Автоматизируйте приём оплаты, выдачу ключей и напоминания о продлении. Бот работает с несколькими серверами и имеет удобный пользовательский интерфейс.

## ✨ Возможности

### 👤 Для пользователей
- **Верификация** через пригласительную ссылку или подтверждение администратором.
- **Просмотр и копирование** ключей подключения (до 3 серверов).
- **Встроенный Speedtest** — проверка пинга и скорости до серверов (Нидерланды, Австрия, Швеция).
- **Прогресс-бар подписки** и дата окончания в формате `ДД.ММ.ГГГГ`.
- **Оплата через СБП** с QR-кодом и уведомлением администратора.
- **Кнопка поддержки** с прямой ссылкой на администратора.

### 🛠 Для администратора
- **Панель управления** с кнопками, без команд.
- **Статистика:** активные подписки, истекающие, платежи.
- **Управление пользователями:** список, продление подписки.
- **Установка ника и ссылок** для каждого пользователя (3 сервера).
- **Генерация инвайт-кодов** с защитой от подбора (блокировка на 1 час после 3 неверных попыток).
- **Настройка контента:** инструкция, реквизиты, QR-код, ссылки на приложения.
- **Тестирование напоминаний** командой `/testremind`.
- **Автоматическая отправка уведомлений** за 3 дня, 1 день и в день окончания подписки.

## 📋 Требования

- **Python 3.10+**
- **SQLite** (уже встроен в Python)
- **Библиотеки:** см. `requirements.txt`
- **Сервер** с доступом к `api.telegram.org` (если блокировки — настройте прокси)
- **Три VPS** для VPN-серверов (Нидерланды, Австрия, Швеция) с настроенным Nginx и тестовым файлом для Speedtest

## 🚀 Быстрый старт

### 1. Клонируйте репозиторий

`bash
git clone https://github.com/Nirmatas/lockbox-vpn-bot.git
cd lockbox-vpn-bot`

## 2. Создайте виртуальное окружение и установите зависимости

`bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt`

3. Настройте переменные окружения
Создайте файл .env в корне проекта:

`env
BOT_TOKEN=ваш_токен_от_BotFather
ADMIN_ID=ваш_telegram_id
SUBSCRIPTION_DAYS=30
PROXY_URL=  # необязательно, socks5://127.0.0.1:9150
BOT_TOKEN — токен бота из @BotFather`

ADMIN_ID — ваш Telegram ID (можно узнать у @userinfobot)
SUBSCRIPTION_DAYS — длительность подписки по умолчанию при продлении
PROXY_URL — если Telegram заблокирован в вашей стране

4. Инициализируйте базу данных
База создастся автоматически при первом запуске бота.

5. Запустите бота
   
`bash
python main.py`

Для продакшена рекомендуется использовать systemd-сервис (см. ниже).

⚙️ Настройка серверов для Speedtest
На каждом VPS выполните:

# Разрешить пинг

`iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
apt update && apt install iptables-persistent -y
netfilter-persistent save`

# Создать тестовый файл

`mkdir -p /var/www/speedtest
dd if=/dev/zero of=/var/www/speedtest/speedtest.bin bs=1M count=10`

# Настроить Nginx

`cat > /etc/nginx/sites-available/speedtest <<EOF
server {
    listen 8080;
    server_name ваш_ip_сервера;
    location /speedtest.bin {
        root /var/www/speedtest;
        add_header Cache-Control no-store;
    }
}
EOF`

`ln -s /etc/nginx/sites-available/speedtest /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx`

# Проверить доступность

`curl -I http://ваш_ip_сервера:8080/speedtest.bin`

После настройки всех серверов укажите их IP и URL в файле handlers/speedtest.py.

🖥 Настройка systemd (для автоматического запуска)
Создайте файл `/etc/systemd/system/vpnbot.service:`

ini
[Unit]
Description=VPN Telegram Bot
After=network.target

[Service]
User=root
WorkingDirectory=/root/Bot
EnvironmentFile=/root/Bot/.env
ExecStart=/root/Bot/.venv/bin/python /root/Bot/main.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

Затем выполните:

`bash
systemctl daemon-reload
systemctl enable vpnbot.service
systemctl start vpnbot.service`

🤝 Вклад в проект
Если у вас есть идеи по улучшению — создавайте issue или pull request. Буду рад любой помощи!
