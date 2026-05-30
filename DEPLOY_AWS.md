# AWS EC2 deploy

Bu bot web server emas, Telegram polling bilan ishlaydi. Shuning uchun EC2 instance yetarli.

## 1. EC2 tayyorlash

- Ubuntu 24.04 LTS yoki 22.04 LTS
- Instance: `t3.micro` yoki `t2.micro`
- Security Group: faqat SSH `22` portini o'zingizning IP manzilingizga oching
- Bot uchun inbound HTTP/HTTPS kerak emas

## 2. Serverga kirish

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

## 3. Paketlar

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip rsync
```

## 4. Loyihani joylash

Loyihani serverdagi `/opt/kinobot` papkasiga ko'chiring:

```bash
sudo mkdir -p /opt/kinobot
sudo chown ubuntu:ubuntu /opt/kinobot
rsync -av --exclude ".venv" --exclude "__pycache__" --exclude ".env" ./ ubuntu@YOUR_EC2_PUBLIC_IP:/opt/kinobot/
```

Serverda virtual muhit yarating:

```bash
cd /opt/kinobot
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
```

## 5. Environment

Serverda `.env` yarating:

```bash
nano /opt/kinobot/.env
```

Namuna:

```env
BOT_TOKEN=BotFather_bergan_token
ADMIN_IDS=8539657937
CHANNEL_USERNAME=@askafilmlar
REQUIRED_CHANNELS=@askafilmlar
CODES_CHANNEL_URL=https://t.me/askafilmlar
PROMO_CHANNEL_USERNAME=@Top_Heshtegch
POST_CHANNEL_ID=-1003728451117
INSTAGRAM_URL=https://www.instagram.com/asl.kinolaruz
DB_PATH=/opt/kinobot/data/kinobot.sqlite3
```

## 6. Service yoqish

```bash
sudo cp /opt/kinobot/deploy/kinobot.service /etc/systemd/system/kinobot.service
sudo systemctl daemon-reload
sudo systemctl enable --now kinobot
sudo systemctl status kinobot
```

Loglarni ko'rish:

```bash
journalctl -u kinobot -f
```

Restart:

```bash
sudo systemctl restart kinobot
```

## Muhim

- `.env` ichidagi tokenni hech qachon GitHub yoki ommaviy joyga yuklamang.
- Hozir lokal `.env` ichida haqiqiy token bor. BotFather orqali tokenni rotate qilish tavsiya qilinadi.
- SQLite baza `/opt/kinobot/data/kinobot.sqlite3` da turadi. EC2 backup yoqilmasa, instance/disk muammosida ma'lumot yo'qolishi mumkin.
- Kamida EBS snapshot backup qo'ying. Katta loyiha bo'lsa, bazani Postgres/RDS ga o'tkazish yaxshiroq.
