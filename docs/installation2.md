Nirvor Communication Bangladesh আইএসপি হিসেবে বেশ নাম করেছে। ঢাকা নর্থ জোনে মূলতঃ তাদের নেটওয়ার্ক। বর্তমানে মিরপুর এবং উত্তরায় দুটো পপ, প্রায় ৫ হাজার কাস্টমার। তারা সিদ্ধান্ত নিয়েছে এক্সেল শিট/গুগল শিট এবং হাতে আঁকা ডায়াগ্রাম ছেড়ে একটা প্রপার Network Source of Truth সিস্টেম বানাবে। তাদের লক্ষ্য পরের দুই বছরে ৫০ হাজার কাস্টমারে পৌঁছানো, আর তারপর ১ লক্ষ। এজন্য দরকার একটা শক্ত ফাউন্ডেশন। সেই ফাউন্ডেশনের নাম Nautobot 3.0।

এই চ্যাপ্টারে আমরা দেখব কীভাবে Nirvor Communication Bangladesh তাদের নেটওয়ার্কের জন্য Nautobot 3.0 ইনস্টল এবং সেটআপ করেছে। আপনিও একইভাবে আপনার ISP-এর জন্য সেটআপ করতে পারবেন।

### কেন Nautobot 3.0?

প্রথম প্রশ্ন - কেন Nautobot 3.0? কেন Nautobot 2.x না?

**Nautobot 3.0-এর মূল সুবিধা:**

১. **Flexible Location Hierarchy:** আগে Sites একটা ফ্ল্যাট স্ট্রাকচার ছিল। এখন Locations দিয়ে যত খুশি লেভেলের hierarchy বানাতে পারবেন। Zone → Cluster → POP → Building → Floor - যেভাবে চান সাজান।

২. **Unified Role System:** ডিভাইস রোল, র‍্যাক রোল, IPAM রোল - সব আলাদা আলাদা ছিল। এখন একটা unified Roles system। এক জায়গা থেকে সব ম্যানেজ করুন।

৩. **Custom Relationships:** দুটো যেকোনো অবজেক্টের মধ্যে custom relationship তৈরি করতে পারবেন। যেমন "Device supports Customer" - এরকম relationship।

৪. **Better Performance:** ডেটাবেস optimization, faster queries, improved UI responsiveness।

৫. **GraphQL API:** REST API-এর পাশাপাশি GraphQL support। জটিল query সহজে করা যায়।

৬. **Plugin Ecosystem:** আরো শক্তিশালী plugin system। Golden Config, Device Lifecycle Management - এসব plugin সহজে integrate করা যায়।

Nirvor Communication Bangladesh এই কারণে 3.0 বেছে নিয়েছে। তারা জানে ভবিষ্যতে scaling করতে হবে, তাই শুরু থেকেই সবচেয়ে আধুনিক ভার্সন দিয়ে শুরু করছে।

### সিস্টেম রিকোয়ারমেন্ট

Nautobot চালাতে হলে একটা লিনাক্স সার্ভার দরকার। Nirvor Communication Bangladesh-এর ক্ষেত্রে তারা একটা ডেডিকেটেড Ubuntu server সেটআপ করেছে।

#### হার্ডওয়্যার রিকোয়ারমেন্ট (Nirvor Communication-এর সাইজের জন্য):

```
CPU: 4 cores (Intel/AMD)
RAM: 8 GB (minimum), 16 GB (recommended)
Storage: 100 GB SSD
Network: 1 Gbps ethernet
```

যদি আপনার নেটওয়ার্ক খুব ছোট হয় (১-২ হাজার কাস্টমার), তাহলে 2 cores এবং 4 GB RAM দিয়েও চলবে। কিন্তু future-proofing-এর জন্য একটু বেশি নেওয়া ভালো।

#### সফটওয়্যার রিকোয়ারমেন্ট:

```
Operating System: Ubuntu 24.04 LTS (recommended)
                  অথবা Ubuntu 22.04 LTS
                  অথবা Debian 12

Python: 3.12 বা তার উপরে
PostgreSQL: 13 বা তার উপরে
Redis: 6.0 বা তার উপরে
```

Nirvor Communication Ubuntu 24.04 LTS বেছে নিয়েছে কারণ এটা long-term support পাবে ২০২৯ সাল পর্যন্ত।

### ইনস্টলেশন পদ্ধতি - কোনটা বেছে নেবেন?

Nautobot ইনস্টল করার দুটো প্রধান উপায়:

**১. Docker + Poetry দিয়ে (সহজ, দ্রুত, Production-Ready)**

- ১৫-২০ মিনিটে চালু করা যায়
- Development এবং Production উভয়ের জন্যই উপযুক্ত
- Plugin management সহজ (Poetry দিয়ে)
- Official `nautobot-docker-compose` repository ব্যবহার করে

**২. Traditional Installation (সম্পূর্ণ নিয়ন্ত্রণ নিজ হাতে)**

- একটু সময় লাগে (৩০-৪৫ মিনিট)
- Bare-metal বা non-Docker environment-এর জন্য
- সব কিছু নিজের মতো সাজানো যায়

Nirvor Communication Bangladesh **পদ্ধতি ১** বেছে নিয়েছে — Docker + Poetry — কারণ এটা plugin management (nautobot-ssot, nautobot-chatops) এবং ভবিষ্যতে version upgrade করার জন্য অনেক বেশি সুবিধাজনক।

আমরা দুটোই দেখব, তবে পদ্ধতি ১-এ বেশি মনোযোগ দেব।

---

### পদ্ধতি ১: Docker + Poetry দিয়ে Production-Ready সেটআপ

Nirvor Communication Bangladesh এই পদ্ধতিটাই production-এ ব্যবহার করেছে। এটা official `nautobot-docker-compose` repository এবং Poetry ব্যবহার করে — plugin management এবং version upgrade অনেক সহজ হয়।

#### অটোমেটেড ডেপ্লয়মেন্ট স্ক্রিপ্ট

Nirvor Communication Bangladesh একটা automated deployment script তৈরি করেছে যা সব কিছু একসাথে করে দেয়:

```bash
# deploy-nautobot.sh ডাউনলোড বা তৈরি করুন
nano deploy-nautobot.sh
chmod +x deploy-nautobot.sh
sudo ./deploy-nautobot.sh
```

এই স্ক্রিপ্ট নিচের সব কাজ করে:
- Docker এবং Docker Compose Plugin ইনস্টল
- Poetry ইনস্টল (Python dependency management)
- `nautobot-docker-compose` repository clone
- সব environment files তৈরি (random credentials সহ)
- Containers build এবং start
- Systemd service তৈরি (boot-এ auto-start)
- Credentials একটা ফাইলে সেভ

স্ক্রিপ্ট শেষ হলে credentials এখানে পাওয়া যাবে:

```bash
cat /opt/nautobot/CREDENTIALS.txt
```

#### ম্যানুয়াল সেটআপ (ধাপে ধাপে)

অটোমেটেড স্ক্রিপ্ট ব্যবহার না করলে, নিচের ধাপগুলো অনুসরণ করুন।

#### স্টেপ ১: সার্ভার প্রস্তুত করা

একটা নতুন Ubuntu 24.04 সার্ভার নিন। Nirvor Communication একটা VM ব্যবহার করেছে তাদের NOC-এর ভেতরে।

SSH করে সার্ভারে লগইন করুন:

```bash
ssh admin@nautobot.nirvor.bd
```

সিস্টেম আপডেট করুন:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl python3 python3-pip openssl
```

#### স্টেপ ২: Docker ইনস্টল করা

Docker এবং Docker Compose Plugin ইনস্টল করুন:

```bash
# Docker এর official GPG key যোগ করুন
sudo apt install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Repository যোগ করুন
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker ইনস্টল করুন
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Current user-কে docker group-এ যোগ করুন
sudo usermod -aG docker $USER
sudo systemctl enable docker && sudo systemctl start docker
```

যাচাই করুন:

```bash
docker --version
```

#### স্টেপ ৩: Poetry ইনস্টল করা

Poetry হলো Python dependency management টুল। Nautobot-এর plugin management এবং build process Poetry দিয়ে চলে।

```bash
curl -sSL https://install.python-poetry.org | python3 -

# PATH-এ যোগ করুন
export PATH="$HOME/.local/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

যাচাই করুন:

```bash
poetry --version
```

#### স্টেপ ৪: Nautobot Repository সেটআপ

```bash
sudo mkdir -p /opt/nautobot
sudo chown $USER:$USER /opt/nautobot
cd /opt/nautobot

# Official nautobot-docker-compose repository clone করুন
git clone https://github.com/nautobot/nautobot-docker-compose.git .
```

Invoke configuration:

```bash
cp invoke.example.yml invoke.yml
```

#### স্টেপ ৫: Environment Files কনফিগার করা

`environments/local.env` তৈরি করুন:

```bash
nano environments/local.env
```

```bash
# Nautobot Settings
NAUTOBOT_ALLOWED_HOSTS=*
NAUTOBOT_DEBUG=False
NAUTOBOT_LOG_LEVEL=INFO
NAUTOBOT_TIME_ZONE=Asia/Dhaka

# Database Settings
NAUTOBOT_DB_ENGINE=django.db.backends.postgresql
NAUTOBOT_DB_HOST=db          # গুরুত্বপূর্ণ: 'postgres' নয়, 'db' লিখুন
NAUTOBOT_DB_PORT=5432
NAUTOBOT_DB_NAME=nautobot
NAUTOBOT_DB_USER=nautobot
NAUTOBOT_DB_TIMEOUT=300

# Redis Settings
NAUTOBOT_REDIS_HOST=redis
NAUTOBOT_REDIS_PORT=6379

# Celery Settings — Redis password থাকলে URL-এ %40 দিয়ে @ encode করুন
NAUTOBOT_CELERY_BROKER_URL=redis://:your_redis_password@redis:6379/0
NAUTOBOT_CELERY_RESULT_BACKEND=redis://:your_redis_password@redis:6379/0
NAUTOBOT_CACHES_LOCATION=redis://:your_redis_password@redis:6379/1

# Superuser
NAUTOBOT_CREATE_SUPERUSER=true
NAUTOBOT_SUPERUSER_NAME=admin
NAUTOBOT_SUPERUSER_EMAIL=admin@Nirvor.bd
```

> ⚠️ **সতর্কতা:** Redis password-এ `@` চিহ্ন থাকলে connection URL-এ `%40` দিয়ে replace করতে হবে। যেমন `nirvorn@Redis2025$` হবে `nirvor%40Redis2025$`। নাহলে Celery connect করতে পারবে না।

`environments/creds.env` তৈরি করুন (এই ফাইল কখনো Git-এ commit করবেন না!):

```bash
nano environments/creds.env
```

```bash
# Secret Key — এটা generate করুন:
# python3 -c "import secrets; print(secrets.token_urlsafe(50))"
NAUTOBOT_SECRET_KEY=your_generated_secret_key

# Database Password
NAUTOBOT_DB_PASSWORD=your_strong_db_password
POSTGRES_PASSWORD=your_strong_db_password
POSTGRES_USER=nautobot
POSTGRES_DB=nautobot

# Superuser Password
NAUTOBOT_SUPERUSER_PASSWORD=your_strong_admin_password
NAUTOBOT_SUPERUSER_API_TOKEN=your_40_char_hex_token

# Redis Password
NAUTOBOT_REDIS_PASSWORD=your_redis_password
```

ফাইল দুটো secure করুন:

```bash
chmod 0600 environments/creds.env
chmod 0600 environments/local.env
```

#### স্টেপ ৬: Python Version এবং Nautobot Version সেট করা

```bash
export PYTHON_VER=3.12
export NAUTOBOT_VERSION=3.0.6
```

এই দুটো variable পরবর্তী সব build command-এ দরকার হবে।

#### স্টেপ ৭: Poetry Dependencies ইনস্টল করা

```bash
cd /opt/nautobot
poetry install --no-interaction
```

#### স্টেপ ৮: Containers Build করা

```bash
export PYTHON_VER=3.12
export NAUTOBOT_VERSION=3.0.6

docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  build --no-cache
```

এটা ১০-২০ মিনিট লাগতে পারে।

#### স্টেপ ৯: Nautobot চালু করা

```bash
docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  up -d
```

স্ট্যাটাস চেক করুন:

```bash
docker ps
```

দেখাবে:

```
nautobot-docker-compose-nautobot-1      Up (healthy)
nautobot-docker-compose-db-1           Up (healthy)
nautobot-docker-compose-redis-1        Up
nautobot-docker-compose-celery_worker-1 Up
nautobot-docker-compose-celery_beat-1  Up
```

Logs দেখুন:

```bash
docker logs nautobot-docker-compose-nautobot-1 --tail 50
```

#### স্টেপ ১০: প্রথম লগইন

ব্রাউজারে যান:

```
http://your-server-ip:8080
```

Nirvor Communication-এর ক্ষেত্রে:

```
http://10.10.0.36:8080
```

`creds.env`-এ দেওয়া admin username ও password দিয়ে লগইন করুন।

অভিনন্দন! Nautobot 3.0.6 চালু হয়ে গেছে।

#### সাধারণ Management Commands

```bash
cd /opt/nautobot

# Containers restart করুন
docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml restart

# Logs দেখুন (live)
docker logs -f nautobot-docker-compose-nautobot-1

# Nautobot shell-এ যান
docker exec -it nautobot-docker-compose-nautobot-1 nautobot-server shell

# নতুন superuser তৈরি করুন
docker exec -it nautobot-docker-compose-nautobot-1 \
  nautobot-server createsuperuser --username newuser --email user@Nirvor.bd

# Database backup নিন
docker exec nautobot-docker-compose-db-1 \
  pg_dump -U nautobot nautobot > nautobot_backup_$(date +%Y%m%d).sql
```

---

### পদ্ধতি ২: Traditional Installation (Bare-Metal)

এই পদ্ধতিতে Docker ছাড়া সরাসরি সার্ভারে Nautobot ইনস্টল করা হয়। Docker ব্যবহারের সুযোগ না থাকলে বা সম্পূর্ণ নিয়ন্ত্রণ চাইলে এই পদ্ধতি ব্যবহার করুন।

https://docs.nautobot.com/projects/core/en/stable/user-guide/administration/installation/nautobot/

---

### Plugin ইনস্টলেশন

Nautobot চালু হওয়ার পর Nirvor Communication Bangladesh দুটো গুরুত্বপূর্ণ plugin যোগ করেছে।

#### Docker পদ্ধতিতে Plugin যোগ করা (Poetry)

Docker + Poetry পদ্ধতিতে plugin যোগ করতে `poetry add` ব্যবহার করুন:

```bash
cd /opt/nautobot

# nautobot-ssot (Single Source of Truth integration framework)
poetry add nautobot-ssot[all]

# nautobot-chatops (Slack/Teams integration)
poetry add nautobot-chatops
```

Plugin যোগ করার পর container rebuild করতে হবে:

```bash
export PYTHON_VER=3.12
export NAUTOBOT_VERSION=3.0.6

docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  build --no-cache

docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  up -d
```

#### nautobot_config.py-তে Plugin Enable করা

`/opt/nautobot/config/nautobot_config.py` ফাইলে PLUGINS list-এ যোগ করুন:

```python
PLUGINS = [
    "nautobot_ssot",
    "nautobot_chatops",
]

PLUGINS_CONFIG = {
    "nautobot_ssot": {
        # SSoT configuration
    },
    "nautobot_chatops": {
        # Chatops configuration (Slack token ইত্যাদি)
    },
}
```

Containers restart করুন:

```bash
docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  restart
```

