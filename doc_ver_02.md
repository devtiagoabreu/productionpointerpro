# **DOCUMENTAÇÃO COMPLETA - PRODUCTION POINTER PRO**
## **Instalação em Servidor Físico para Ambiente Industrial Têxtil**

---

## **ÍNDICE GERAL**

1. [VISÃO GERAL DO SISTEMA](#1-visão-geral-do-sistema)
2. [ARQUITETURA DE SERVIDOR FÍSICO](#2-arquitetura-de-servidor-físico)
3. [PRÉ-REQUISITOS DE HARDWARE E SOFTWARE](#3-pré-requisitos-de-hardware-e-software)
4. [INSTALAÇÃO PASSO A PASSO](#4-instalação-passo-a-passo)
5. [CONFIGURAÇÃO DE REDE INDUSTRIAL](#5-configuração-de-rede-industrial)
6. [MODELO DE DADOS COMPLETO](#6-modelo-de-dados-completo)
7. [FLUXOS DE TRABALHO DETALHADOS](#7-fluxos-de-trabalho-detalhados)
8. [INTEGRAÇÃO COM ERP SYSTÊXTIL](#8-integração-com-erp-systêxtil)
9. [SEGURANÇA E HARDENING](#9-segurança-e-hardening)
10. [BACKUP E RECUPERAÇÃO DE DESASTRES](#10-backup-e-recuperação-de-desastres)
11. [MANUTENÇÃO E MONITORAMENTO](#11-manutenção-e-monitoramento)
12. [EXPANSÕES FUTURAS](#12-expansões-futuras)

---

## **1. VISÃO GERAL DO SISTEMA**

### **1.1 Contexto da Indústria Têxtil**
Sistema desenvolvido especificamente para **maquinário de tecelagem e beneficiamento têxtil**:
- **Urdideira**: Preparação dos fios
- **Enroladeira**: Enrolamento dos tecidos
- **Revisadeira**: Inspeção de qualidade
- **Jigger**: Tingimento descontínuo
- **Turbo**: Tingimento rápido
- **Stork**: Estamparia
- **Rama**: Tingimento contínuo
- **Calandra**: Acabamento com calor/pressão
- **Sanforizadeira**: Estabilização dimensional
- **Secadeira**: Secagem de tecidos

### **1.2 Problema Atual**
O ERP Systêxtil possui funcionalidade de apontamento mas:
- Telas não são intuitivas para operadores
- Dificuldade de uso em dispositivos móveis
- Falta de feedback visual imediato
- Complexidade para operadores com pouca familiaridade digital

### **1.3 Solução Proposta**
Sistema web complementar ao ERP com:
- Interface mobile-first
- Controle por QR Codes
- Cálculo automático de eficiências
- Painéis visuais em tempo real
- Foco na metodologia LEAN

---

## **2. ARQUITETURA DE SERVIDOR FÍSICO**

### **2.1 Diagrama de Arquitetura Física**
```
┌─────────────────────────────────────────────────────────────────┐
│                    SERVIDOR FÍSICO DEDICADO                     │
├─────────────────────────────────────────────────────────────────┤
│  Ubuntu Server 22.04 LTS                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  SERVIÇOS INSTALADOS                    │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │    │
│  │  │PostgreSQL│  │  Redis   │  │  Python  │  │ Nginx  │  │    │
│  │  │  15.4    │  │   7.2    │  │  3.11    │  │ 1.24   │  │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └────────┘  │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │    │
│  │  │Supervisor│  │ Fail2ban │  │   UFW    │  │Cron    │  │    │
│  │  │  4.2     │  │  0.11    │  │  Firewall│  │Jobs    │  │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └────────┘  │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────┬───────────────────────────────────────────────┘
                  │
    ┌─────────────┼─────────────────────────────────────┐
    │ Rede        │ Rede                                │
    │ Industrial  │ Administrativa                      │
    │ 10.0.0.0/24 │ 192.168.1.0/24                     │
    │             │                                     │
    │  ┌──────────▼──────────┐  ┌──────────▼──────────┐ │
    │  │   SWITCH INDUSTRIAL │  │   ROTEADOR/FIREWALL │ │
    │  │      (GERENCIÁVEL)  │  │                     │ │
    │  └──────────┬──────────┘  └──────────┬──────────┘ │
    │             │                         │           │
┌───▼─────────────▼──────────┐   ┌─────────▼────────────▼──┐
│    CHÃO DE FÁBRICA         │   │      ESCRITÓRIO         │
│  ┌─────────────────────┐   │   │  ┌──────────────────┐   │
│  │ Access Points Wi-Fi │◀──┘   │  │   ERP Systêxtil  │   │
│  │    (10.0.0.10-20)   │       │  │     Servidor     │   │
│  └──────────┬──────────┘       │  └─────────┬────────┘   │
│             │                  │            │            │
│  ┌──────────▼──────────┐       │  ┌─────────▼────────┐   │
│  │  Dispositivos Mobile│       │  │   Desktops       │   │
│  │  • Celulares Android│       │  │  Admin/Supervis. │   │
│  │  • Tablets          │       │  └──────────────────┘   │
│  │  • Computadores     │       │                         │
│  └─────────────────────┘       └─────────────────────────┘
```

### **2.2 Estrutura de Diretórios no Servidor**
```
/
├── opt/
│   └── productionpointer/              # Raiz da aplicação
│       ├── app/                        # Código fonte Flask
│       │   ├── __init__.py
│       │   ├── models.py
│       │   ├── routes/
│       │   ├── templates/
│       │   ├── static/
│       │   ├── config.py
│       │   └── requirements.txt
│       │
│       ├── data/                       # Dados persistentes
│       │   ├── postgres/               # Volume PostgreSQL
│       │   │   ├── base/
│       │   │   ├── pg_wal/
│       │   │   └── pg_log/
│       │   │
│       │   ├── redis/                  # Dados Redis
│       │   │   └── dump.rdb
│       │   │
│       │   ├── backups/                # Backups automáticos
│       │   │   ├── diarios/
│       │   │   ├── semanais/
│       │   │   └── mensais/
│       │   │
│       │   ├── logs/                   # Logs da aplicação
│       │   │   ├── app/
│       │   │   ├── nginx/
│       │   │   ├── postgresql/
│       │   │   └── supervisor/
│       │   │
│       │   └── uploads/                # Uploads de usuários
│       │       ├── qrcodes/            # QR Codes gerados
│       │       ├── photos/             # Fotos de paradas
│       │       └── reports/            # Relatórios exportados
│       │
│       ├── venv/                       # Ambiente virtual Python
│       │   ├── bin/
│       │   ├── lib/
│       │   └── include/
│       │
│       ├── scripts/                    # Scripts de administração
│       │   ├── install.sh
│       │   ├── backup.sh
│       │   ├── restore.sh
│       │   ├── update.sh
│       │   ├── health_check.sh
│       │   └── maintenance.sh
│       │
│       └── config/                     # Configurações do sistema
│           ├── nginx/
│           ├── supervisor/
│           ├── systemd/
│           └── cron/
│
├── etc/
│   ├── nginx/
│   │   └── sites-available/productionpointer
│   ├── supervisor/
│   │   └── conf.d/productionpointer.conf
│   ├── fail2ban/
│   │   └── jail.d/productionpointer.conf
│   └── cron.d/
│       └── productionpointer
│
└── var/
    └── log/
        ├── productionpointer/
        ├── nginx/
        ├── postgresql/
        └── supervisor/
```

### **2.3 Especificações Técnicas do Servidor**

#### **Hardware Mínimo Recomendado**
```
PROCESSADOR: Intel Xeon E-2234 ou equivalente (4 núcleos/8 threads)
MEMÓRIA RAM: 16GB DDR4 ECC (expandível para 32GB)
ARMAZENAMENTO: 2x 480GB SSD RAID 1 (para sistema e dados)
              1x 2TB HDD (para backups)
REDE: 2x Gigabit Ethernet (Intel I350)
FONTE: 500W Redundante (ou UPS conectado)
```

#### **Hardware Ideal para 100+ Dispositivos**
```
PROCESSADOR: Intel Xeon Silver 4210 (10 núcleos/20 threads)
MEMÓRIA RAM: 32GB DDR4 ECC
ARMAZENAMENTO: RAID 10 com 4x 960GB SSD NVMe
BACKUP: 4TB HDD + Fita LTO (opcional)
REDE: 2x 10GbE SFP+ (para futura expansão)
ENERGIA: UPS 1500VA + Gerador
```

---

## **3. PRÉ-REQUISITOS DE HARDWARE E SOFTWARE**

### **3.1 Checklist de Pré-Instalação**

#### **Infraestrutura Física**
- [ ] Servidor rack 1U ou 2U instalado
- [ ] Rack com ventilação adequada
- [ ] Sistema de refrigeração (temperatura < 25°C)
- [ ] UPS (No-break) com autonomia mínima de 30 minutos
- [ ] Cabos de rede CAT6 ou superior
- [ ] Switch gerenciável 24 portas (pelo menos)
- [ ] Access Points Wi-Fi industriais (resistente a poeira/umidade)
- [ ] Impressora para QR Codes (Zebra ou similar)

#### **Rede e Conectividade**
- [ ] IP fixo do servidor definido
- [ ] Domínio registrado (ex: producao.empresa.com.br)
- [ ] Certificado SSL (Let's Encrypt ou comercial)
- [ ] VLAN segregada para equipamentos industriais
- [ ] QoS configurado para priorizar tráfego de produção
- [ ] Firewall com regras específicas

#### **Preparação do Ambiente**
- [ ] Backup do ERP atual
- [ ] Documentação das APIs do Systêxtil
- [ ] Lista de máquinas com códigos
- [ ] Estrutura de produtos (nível.grupo.subgrupo.item)
- [ ] Roteiros de produção por produto
- [ ] Velocidades padrão por máquina e produto

### **3.2 Software Base Necessário**

#### **Sistema Operacional**
```bash
# Ubuntu Server 22.04 LTS (recomendado para suporte a 5 anos)
# Ou CentOS 7/8 para ambientes mais corporativos

# Verificar compatibilidade
uname -m  # Deve retornar x86_64
lsb_release -a  # Verificar versão do Ubuntu
```

#### **Pacotes Essenciais**
```bash
# Lista completa de dependências do sistema
ESSENTIAL_PACKAGES="
git build-essential curl wget vim htop
python3.11 python3.11-venv python3.11-dev
libpq-dev libssl-dev libffi-dev
nginx postgresql postgresql-contrib
redis-server supervisor fail2ban
ufw unattended-upgrades
logrotate rsync borgbackup
net-tools tcpdump iftop
certbot python3-certbot-nginx
"

# Para impressoras Zebra (QR Codes)
ZEBRA_PACKAGES="
cups cups-bsd cups-client
printer-driver-zebra
samba samba-client
```

---

## **4. INSTALAÇÃO PASSO A PASSO**

### **4.1 Fase 1: Preparação do Sistema Operacional**

#### **4.1.1 Instalação do Ubuntu Server**
```bash
# 1. Baixar imagem do site oficial
# Ubuntu Server 22.04.3 LTS - 64-bit

# 2. Criar USB bootável (usando balenaEtcher ou Rufus)

# 3. Instalação com particionamento manual:
# /     - 50GB  (ext4)
# /boot - 1GB   (ext4)
# /opt  - 400GB (ext4)  # Para aplicação e dados
# swap  - 16GB  (swap)  # Igual à RAM

# 4. Durante instalação:
# • Hostname: production-server
# • Usuário: admin-prod
# • Instalar OpenSSH server (IMPORTANTE)
# • Instalar PostgreSQL, Redis (opcional, faremos manual)
```

#### **4.1.2 Configuração Inicial Pós-Instalação**
```bash
#!/bin/bash
# initial_setup.sh

# Atualizar sistema
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y

# Configurar timezone para América/São Paulo
sudo timedatectl set-timezone America/Sao_Paulo

# Instalar NTP para sincronização de tempo
sudo apt install -y chrony
sudo systemctl enable chrony
sudo systemctl start chrony

# Desabilitar IPv6 se não for usado (melhora performance)
echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
echo "net.ipv6.conf.lo.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Otimizar parâmetros do kernel para PostgreSQL
echo "# Otimizações para PostgreSQL e rede" | sudo tee -a /etc/sysctl.conf
echo "kernel.shmmax = 68719476736" | sudo tee -a /etc/sysctl.conf
echo "kernel.shmall = 4294967296" | sudo tee -a /etc/sysctl.conf
echo "net.core.rmem_max = 16777216" | sudo tee -a /etc/sysctl.conf
echo "net.core.wmem_max = 16777216" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.tcp_rmem = 4096 87380 16777216" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.tcp_wmem = 4096 65536 16777216" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Configurar limites do sistema
echo "* soft nofile 65536" | sudo tee -a /etc/security/limits.conf
echo "* hard nofile 65536" | sudo tee -a /etc/security/limits.conf
echo "* soft nproc 65536" | sudo tee -a /etc/security/limits.conf
echo "* hard nproc 65536" | sudo tee -a /etc/security/limits.conf
echo "postgres soft nofile 65536" | sudo tee -a /etc/security/limits.conf
echo "postgres hard nofile 65536" | sudo tee -a /etc/security/limits.conf
```

### **4.2 Fase 2: Instalação dos Serviços Base**

#### **4.2.1 PostgreSQL 15 (Banco de Dados)**
```bash
#!/bin/bash
# install_postgresql.sh

# Adicionar repositório oficial do PostgreSQL
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update

# Instalar PostgreSQL 15
sudo apt install -y postgresql-15 postgresql-contrib-15
sudo apt install -y postgresql-15-postgis-3 pgtop

# Configurar PostgreSQL
sudo -i -u postgres

# Criar diretório para dados no /opt
sudo mkdir -p /opt/productionpointer/data/postgres
sudo chown -R postgres:postgres /opt/productionpointer/data/postgres

# Parar PostgreSQL para mover dados
sudo systemctl stop postgresql

# Mover dados do PostgreSQL para /opt (opcional, para SSD dedicado)
sudo mv /var/lib/postgresql/15/main /opt/productionpointer/data/postgres/
sudo ln -s /opt/productionpointer/data/postgres/main /var/lib/postgresql/15/main

# Ajustar configurações
sudo nano /etc/postgresql/15/main/postgresql.conf

# Alterar as seguintes linhas:
# listen_addresses = 'localhost,10.0.0.1'  # IP interno da fábrica
# port = 5432
# max_connections = 200
# shared_buffers = 4GB                    # 25% da RAM
# effective_cache_size = 12GB             # 75% da RAM
# work_mem = 64MB
# maintenance_work_mem = 1GB
# checkpoint_completion_target = 0.9
# wal_buffers = 16MB
# default_statistics_target = 100
# random_page_cost = 1.1
# effective_io_concurrency = 200
# max_wal_size = 2GB
# min_wal_size = 1GB

# Configurar pg_hba.conf para acesso da aplicação
sudo nano /etc/postgresql/15/main/pg_hba.conf
# Adicionar linha:
# host    all             all             10.0.0.0/24            md5

# Reiniciar PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Criar banco e usuário
sudo -u postgres psql << EOF
CREATE DATABASE production_pointer;
CREATE USER production_user WITH PASSWORD 'S3nh@F0rt3!2024';
ALTER DATABASE production_pointer OWNER TO production_user;
GRANT ALL PRIVILEGES ON DATABASE production_pointer TO production_user;
\c production_pointer
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "postgis";
\q
EOF
```

#### **4.2.2 Redis 7 (Cache e WebSockets)**
```bash
#!/bin/bash
# install_redis.sh

# Instalar Redis
sudo apt install -y redis-server

# Configurar Redis
sudo nano /etc/redis/redis.conf

# Alterar configurações:
# bind 127.0.0.1 10.0.0.1
# protected-mode yes
# port 6379
# requirepass "R3d!sS3nh@2024"
# maxmemory 2gb
# maxmemory-policy allkeys-lru
# save 900 1
# save 300 10
# save 60 10000
# dir /opt/productionpointer/data/redis

# Criar diretório para dados
sudo mkdir -p /opt/productionpointer/data/redis
sudo chown -R redis:redis /opt/productionpointer/data/redis

# Habilitar como serviço systemd
sudo systemctl restart redis-server
sudo systemctl enable redis-server

# Testar conexão
redis-cli -a "R3d!sS3nh@2024" ping
```

#### **4.2.3 Python 3.11 e Ambiente Virtual**
```bash
#!/bin/bash
# install_python.sh

# Instalar Python 3.11
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.11 python3.11-venv python3.11-dev
sudo apt install -y python3-pip

# Criar usuário para a aplicação
sudo useradd -m -s /bin/bash -d /opt/productionpointer production
sudo usermod -a -G postgres production

# Criar estrutura de diretórios
sudo mkdir -p /opt/productionpointer/{app,data,venv,scripts,config}
sudo chown -R production:production /opt/productionpointer

# Criar ambiente virtual
sudo -u production python3.11 -m venv /opt/productionpointer/venv

# Instalar pip e setuptools atualizados
sudo -u production /opt/productionpointer/venv/bin/pip install --upgrade pip setuptools wheel
```

### **4.3 Fase 3: Instalação da Aplicação**

#### **4.3.1 Clonar e Configurar o Projeto**
```bash
#!/bin/bash
# install_application.sh

# Como usuário production
sudo -u production -i

# Clonar repositório
cd /opt/productionpointer
git clone https://github.com/devtiagoabreu/productionpointerpro.git app

# Criar arquivo de ambiente
cd app
cp .env.example .env
nano .env  # Configurar com valores reais

# Conteúdo mínimo do .env:
"""
# Configurações do Flask
FLASK_APP=app
FLASK_ENV=production
SECRET_KEY='Ch@v3-S3cr3t@-Muit0-F0rt3-2024!'
DEBUG=False

# Banco de dados
DATABASE_URL=postgresql://production_user:S3nh@F0rt3!2024@localhost:5432/production_pointer

# Redis
REDIS_URL=redis://:R3d!sS3nh@2024@localhost:6379/0

# ERP Integration
ERP_API_URL=https://erp.systextil.com.br/api
ERP_API_KEY=sua-chave-api-erp
ERP_SYNC_INTERVAL=300  # 5 minutos

# Configurações da aplicação
TIMEZONE=America/Sao_Paulo
QRCODE_SIZE=10
UPLOAD_FOLDER=/opt/productionpointer/data/uploads
MAX_CONTENT_LENGTH=16777216  # 16MB

# E-mail (para notificações)
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=True
MAIL_USERNAME=notificacoes@empresa.com.br
MAIL_PASSWORD=senha-email

# Logging
LOG_LEVEL=INFO
LOG_FILE=/opt/productionpointer/data/logs/app.log
"""

# Instalar dependências Python
/opt/productionpointer/venv/bin/pip install -r requirements.txt

# Lista de pacotes Python essenciais (requirements.txt):
"""
Flask==2.3.3
Flask-SQLAlchemy==3.0.5
Flask-Migrate==4.0.5
Flask-JWT-Extended==4.5.3
Flask-CORS==4.0.0
Flask-SocketIO==5.3.4
Flask-Limiter==3.3.3
psycopg2-binary==2.9.7
redis==4.6.0
celery==5.3.1
requests==2.31.0
python-dotenv==1.0.0
gunicorn==21.2.0
qrcode[pil]==7.4.2
pyzbar==0.1.9
Pillow==10.0.0
pandas==2.1.0
openpyxl==3.1.2
reportlab==4.0.4
python-dateutil==2.8.2
pytz==2023.3
jsonschema==4.19.0
argon2-cffi==23.1.0
"""

# Testar instalação
/opt/productionpointer/venv/bin/python -c "import flask; print('Flask instalado corretamente')"
```

#### **4.3.2 Configurar Gunicorn (WSGI Server)**
```bash
#!/bin/bash
# configure_gunicorn.sh

# Criar arquivo de configuração do Gunicorn
sudo tee /opt/productionpointer/config/gunicorn.conf.py << 'EOF'
import multiprocessing

# Configurações do Gunicorn
bind = "127.0.0.1:8000"
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = "gevent"
worker_connections = 1000
timeout = 120
keepalive = 5
max_requests = 1000
max_requests_jitter = 50

# Logging
accesslog = "/opt/productionpointer/data/logs/gunicorn_access.log"
errorlog = "/opt/productionpointer/data/logs/gunicorn_error.log"
loglevel = "info"
access_log_format = '%(h)s %(l)s %(u)s %(t)s "%(r)s" %(s)s %(b)s "%(f)s" "%(a)s"'

# Segurança
proc_name = "production_pointer"
user = "production"
group = "production"
EOF

# Testar Gunicorn
cd /opt/productionpointer/app
sudo -u production /opt/productionpointer/venv/bin/gunicorn -c /opt/productionpointer/config/gunicorn.conf.py app:app
```

### **4.4 Fase 4: Configuração do Web Server (Nginx)**

#### **4.4.1 Instalar e Configurar Nginx**
```bash
#!/bin/bash
# configure_nginx.sh

# Instalar Nginx
sudo apt install -y nginx

# Remover configuração padrão
sudo rm -f /etc/nginx/sites-enabled/default

# Criar configuração para o Production Pointer
sudo tee /etc/nginx/sites-available/productionpointer << 'EOF'
# Production Pointer Pro - Configuração Nginx
# Servidor HTTP (redireciona para HTTPS)
server {
    listen 80;
    listen [::]:80;
    server_name producao.empresa.com.br;
    
    # Redirecionar para HTTPS
    return 301 https://$server_name$request_uri;
}

# Servidor HTTPS
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name producao.empresa.com.br;
    
    # SSL/TLS Configuration
    ssl_certificate /etc/letsencrypt/live/producao.empresa.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/producao.empresa.com.br/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;
    ssl_session_tickets off;
    
    # Diffie-Hellman parameters
    ssl_dhparam /etc/nginx/dhparam.pem;
    
    # SSL protocols
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # SSL ciphers
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
    
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
    
    # Logs
    access_log /opt/productionpointer/data/logs/nginx_access.log;
    error_log /opt/productionpointer/data/logs/nginx_error.log;
    
    # Root directory
    root /opt/productionpointer/app/static;
    
    # Proxy para aplicação Flask
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    # Static files
    location /static {
        alias /opt/productionpointer/app/static;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # Uploads
    location /uploads {
        alias /opt/productionpointer/data/uploads;
        internal;
    }
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # File upload size
    client_max_body_size 20M;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript 
               application/javascript application/xml+rss 
               application/json application/octet-stream;
}
EOF

# Habilitar site
sudo ln -s /etc/nginx/sites-available/productionpointer /etc/nginx/sites-enabled/

# Gerar parâmetros Diffie-Hellman (pode demorar)
sudo openssl dhparam -out /etc/nginx/dhparam.pem 2048

# Testar configuração
sudo nginx -t

# Se ok, reiniciar Nginx
sudo systemctl restart nginx
sudo systemctl enable nginx
```

#### **4.4.2 Configurar Certificado SSL (Let's Encrypt)**
```bash
#!/bin/bash
# configure_ssl.sh

# Instalar Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obter certificado SSL
sudo certbot --nginx -d producao.empresa.com.br

# Configurar renovação automática
sudo certbot renew --dry-run

# Adicionar ao crontab para renovação automática
(sudo crontab -l 2>/dev/null; echo "0 12 * * * /usr/bin/certbot renew --quiet") | sudo crontab -
```

### **4.5 Fase 5: Gerenciamento de Processos (Supervisor)**

#### **4.5.1 Configurar Supervisor**
```bash
#!/bin/bash
# configure_supervisor.sh

# Instalar Supervisor
sudo apt install -y supervisor

# Configurar serviço da aplicação
sudo tee /etc/supervisor/conf.d/productionpointer.conf << 'EOF'
[program:productionpointer]
command=/opt/productionpointer/venv/bin/gunicorn -c /opt/productionpointer/config/gunicorn.conf.py app:app
directory=/opt/productionpointer/app
user=production
autostart=true
autorestart=true
startretries=3
startsecs=5
stopwaitsecs=30
stdout_logfile=/opt/productionpointer/data/logs/app_stdout.log
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
stderr_logfile=/opt/productionpointer/data/logs/app_stderr.log
stderr_logfile_maxbytes=50MB
stderr_logfile_backups=10
environment=HOME="/opt/productionpointer",USER="production",PATH="/opt/productionpointer/venv/bin:%(ENV_PATH)s"

[program:celery_worker]
command=/opt/productionpointer/venv/bin/celery -A app.celery worker --loglevel=info --concurrency=4
directory=/opt/productionpointer/app
user=production
autostart=true
autorestart=true
startretries=3
startsecs=10
stopwaitsecs=30
stdout_logfile=/opt/productionpointer/data/logs/celery_worker_stdout.log
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
stderr_logfile=/opt/productionpointer/data/logs/celery_worker_stderr.log
stderr_logfile_maxbytes=50MB
stderr_logfile_backups=10
environment=HOME="/opt/productionpointer",USER="production",PATH="/opt/productionpointer/venv/bin:%(ENV_PATH)s"

[program:celery_beat]
command=/opt/productionpointer/venv/bin/celery -A app.celery beat --loglevel=info
directory=/opt/productionpointer/app
user=production
autostart=true
autorestart=true
startretries=3
startsecs=10
stopwaitsecs=30
stdout_logfile=/opt/productionpointer/data/logs/celery_beat_stdout.log
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
stderr_logfile=/opt/productionpointer/data/logs/celery_beat_stderr.log
stderr_logfile_maxbytes=50MB
stderr_logfile_backups=10
environment=HOME="/opt/productionpointer",USER="production",PATH="/opt/productionpointer/venv/bin:%(ENV_PATH)s"

[group:production]
programs=productionpointer,celery_worker,celery_beat
priority=999
EOF

# Criar diretório de logs
sudo mkdir -p /opt/productionpointer/data/logs
sudo chown -R production:production /opt/productionpointer/data/logs

# Recarregar configurações
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start all

# Verificar status
sudo supervisorctl status
```

### **4.6 Fase 6: Firewall e Segurança**

#### **4.6.1 Configurar Firewall (UFW)**
```bash
#!/bin/bash
# configure_firewall.sh

# Habilitar UFW
sudo ufw --force enable

# Regras básicas
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Permitir SSH (alterar porta se necessário)
sudo ufw allow 22/tcp

# Permitir HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Permitir rede interna da fábrica
sudo ufw allow from 10.0.0.0/24 to any

# Permitir acesso do escritório (se diferente)
sudo ufw allow from 192.168.1.0/24 to any

# Regras específicas para PostgreSQL (apenas rede interna)
sudo ufw allow from 10.0.0.0/24 to any port 5432

# Regras específicas para Redis (apenas localhost)
sudo ufw allow from 127.0.0.1 to any port 6379

# Mostrar regras
sudo ufw status verbose

# Ativar logging
sudo ufw logging on
```

#### **4.6.2 Configurar Fail2ban (Proteção contra ataques)**
```bash
#!/bin/bash
# configure_fail2ban.sh

# Criar jail específico para a aplicação
sudo tee /etc/fail2ban/jail.d/productionpointer.local << 'EOF'
[nginx-http-auth]
enabled = true
port = http,https
filter = nginx-http-auth
logpath = /opt/productionpointer/data/logs/nginx_access.log
maxretry = 3
bantime = 3600
findtime = 600

[nginx-badbots]
enabled = true
port = http,https
filter = nginx-badbots
logpath = /opt/productionpointer/data/logs/nginx_access.log
maxretry = 2
bantime = 86400
findtime = 600

[nginx-noscript]
enabled = true
port = http,https
filter = nginx-noscript
logpath = /opt/productionpointer/data/logs/nginx_access.log
maxretry = 6
bantime = 86400
findtime = 600

[nginx-proxy]
enabled = true
port = http,https
filter = nginx-proxy
logpath = /opt/productionpointer/data/logs/nginx_access.log
maxretry = 3
bantime = 3600
findtime = 600

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
EOF

# Reiniciar Fail2ban
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban

# Verificar status
sudo fail2ban-client status
```

---

## **5. CONFIGURAÇÃO DE REDE INDUSTRIAL**

### **5.1 Estrutura de Rede para Fábrica**
```
                            ┌─────────────────────┐
                            │  INTERNET           │
                            │  (Operadora)        │
                            └──────────┬──────────┘
                                       │
                            ┌──────────▼──────────┐
                            │  ROTEADOR/FIREWALL  │
                            │  192.168.1.1/24     │
                            └──────────┬──────────┘
                                       │
               ┌───────────────────────┼───────────────────────┐
               │                       │                       │
    ┌──────────▼──────────┐ ┌──────────▼──────────┐ ┌──────────▼──────────┐
    │   SWITCH ESCRITÓRIO │ │  SWITCH INDUSTRIAL  │ │    SERVIDOR LOCAL   │
    │     192.168.1.10    │ │     10.0.0.1/24     │ │   production-server │
    └──────────┬──────────┘ └──────────┬──────────┘ │  • eth0:192.168.1.100│
               │                       │            │  • eth1:10.0.0.100   │
    ┌──────────▼──────────┐ ┌──────────▼──────────┐ └─────────────────────┘
    │  Desktops ERP       │ │   ACCESS POINTS     │
    │  • ERP Systêxtil    │ │   Wi-Fi Industrial  │
    │  • Admin/Planning   │ │   • AP-01:10.0.0.10 │
    └─────────────────────┘ │   • AP-02:10.0.0.11 │
                            │   • AP-03:10.0.0.12 │
                            └──────────┬──────────┘
                                       │
                            ┌──────────▼──────────────────────────────┐
                            │         CHÃO DE FÁBRICA                 │
                            │  ┌──────────┐ ┌──────────┐ ┌──────────┐│
                            │  │ Mobile   │ │ Tablet   │ │  Tablet  ││
                            │  │ Android  │ │ Setor A  │ │ Setor B  ││
                            │  │ 10.0.0.50│ │10.0.0.51 │ │10.0.0.52 ││
                            │  └──────────┘ └──────────┘ └──────────┘│
                            │  ┌──────────┐ ┌──────────┐ ┌──────────┐│
                            │  │ Comput.  │ │ Impress. │ │ Leitor   ││
                            │  │ de Chão  │ │ QR Code  │ │ Cód.Barr.││
                            │  │10.0.0.60 │ │10.0.0.61 │ │10.0.0.62 ││
                            │  └──────────┘ └──────────┘ └──────────┘│
                            └────────────────────────────────────────┘
```

### **5.2 Configuração de Interfaces de Rede**
```bash
#!/bin/bash
# configure_network.sh

# Configurar interfaces de rede
sudo tee /etc/netplan/01-netcfg.yaml << 'EOF'
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
        search: [empresa.com.br]
      routes:
        - to: 10.0.0.0/24
          via: 192.168.1.1
    eth1:
      dhcp4: no
      addresses:
        - 10.0.0.100/24
      mtu: 1500
EOF

# Aplicar configuração
sudo netplan apply

# Configurar roteamento estático para rede industrial
sudo ip route add 10.0.0.0/24 dev eth1

# Verificar configuração
ip addr show
ip route show
```

### **5.3 Configuração de Access Points Wi-Fi Industriais**

#### **Requisitos para Wi-Fi Industrial:**
1. **Múltiplos SSIDs** (um para produção, outro para visitantes)
2. **VLAN tagging** para separação de tráfego
3. **Power over Ethernet (PoE)** para facilidade de instalação
4. **Resistência a poeira/umidade** (IP65 ou superior)
5. **Handoff rápido** para movimento entre áreas

#### **Exemplo de configuração (Ubiquiti/Aruba):**
```
SSID Principal: "PRODUCAO_EMPRESA"
• Segurança: WPA2-Enterprise com RADIUS (ou WPA2-PSK para simplicidade)
• VLAN: 10 (produção)
• Banda: 5GHz preferencial (menos interferência)
• Canais: Fixos (1, 6, 11 no 2.4GHz) para evitar interferência
• Potência: Ajustada para cobertura sem sobreposição excessiva
```

---

## **6. MODELO DE DADOS COMPLETO**

### **6.1 Script SQL de Criação do Banco**
```sql
-- production_pointer_schema.sql
-- Executar como: sudo -u postgres psql -d production_pointer -f schema.sql

-- ============================================
-- TABELA: users (Usuários do sistema)
-- ============================================
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    employee_code VARCHAR(20) UNIQUE NOT NULL,        -- Matrícula do funcionário
    username VARCHAR(50) UNIQUE NOT NULL,             -- Login
    password_hash VARCHAR(255) NOT NULL,              -- Hash da senha (Argon2)
    name VARCHAR(100) NOT NULL,                       -- Nome completo
    email VARCHAR(100),
    phone VARCHAR(20),
    
    -- Informações de trabalho
    department VARCHAR(50),                           -- Departamento
    position VARCHAR(50),                             -- Cargo
    shift VARCHAR(20),                                -- Turno (Manhã/Tarde/Noite)
    hire_date DATE,                                   -- Data de admissão
    
    -- Permissões
    user_type VARCHAR(20) NOT NULL DEFAULT 'operator', -- operator, supervisor, admin, planner
    permissions JSONB DEFAULT '{}',                   -- Permissões específicas
    
    -- Status
    is_active BOOLEAN DEFAULT TRUE,
    must_change_password BOOLEAN DEFAULT TRUE,
    last_login TIMESTAMP,
    login_attempts INTEGER DEFAULT 0,
    
    -- Auditoria
    created_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deactivated_at TIMESTAMP,
    
    -- Índices
    INDEX idx_users_username (username),
    INDEX idx_users_employee_code (employee_code),
    INDEX idx_users_department (department),
    INDEX idx_users_is_active (is_active)
);

-- ============================================
-- TABELA: machines (Máquinas/Equipamentos)
-- ============================================
CREATE TABLE machines (
    id SERIAL PRIMARY KEY,
    code VARCHAR(20) UNIQUE NOT NULL,                 -- Código interno da máquina
    name VARCHAR(100) NOT NULL,                       -- Nome/Descrição
    machine_type VARCHAR(50) NOT NULL,                -- urdideira, enroladeira, jigger, etc.
    
    -- Localização
    department VARCHAR(50) NOT NULL,                  -- Setor/Departamento
    line VARCHAR(20),                                 -- Linha de produção
    position INTEGER,                                 -- Posição na linha
    
    -- Características técnicas
    manufacturer VARCHAR(100),                        -- Fabricante
    model VARCHAR(100),                               -- Modelo
    serial_number VARCHAR(50),                        -- Número de série
    installation_date DATE,                           -- Data de instalação
    capacity DECIMAL(10,2),                           -- Capacidade (metros/hora teórica)
    
    -- Status operacional
    status VARCHAR(20) DEFAULT 'stopped',             -- working, stopped, maintenance, setup
    last_production TIMESTAMP,                        -- Última produção
    last_maintenance DATE,                            -- Última manutenção
    
    -- Controle de qualidade
    quality_rating DECIMAL(3,2) DEFAULT 1.0,          -- Avaliação de qualidade (0-1)
    defect_rate DECIMAL(5,2) DEFAULT 0.0,             -- Taxa de defeito histórico
    
    -- QR Code
    qr_code VARCHAR(255) NOT NULL,                    -- Caminho do QR Code
    qr_code_data TEXT,                                -- Dados do QR Code (criptografados)
    
    -- Parâmetros de produção
    efficiency_target DECIMAL(5,2) DEFAULT 85.00,     -- Meta de eficiência (%)
    availability_target DECIMAL(5,2) DEFAULT 95.00,   -- Meta de disponibilidade (%)
    
    -- Auditoria
    created_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Índices
    INDEX idx_machines_code (code),
    INDEX idx_machines_type (machine_type),
    INDEX idx_machines_department (department),
    INDEX idx_machines_status (status)
);

-- ============================================
-- TABELA: products (Produtos - Cadastro completo)
-- ============================================
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    product_code VARCHAR(20) UNIQUE NOT NULL,         -- Formato: 0.00000.000.000000
    description VARCHAR(200) NOT NULL,
    
    -- Hierarquia completa
    level_code VARCHAR(2) NOT NULL,                   -- Nível (0, 1, 2, ...)
    group_code VARCHAR(5) NOT NULL,                   -- Grupo (5 dígitos)
    subgroup_code VARCHAR(3) NOT NULL,                -- Subgrupo (3 dígitos)
    item_code VARCHAR(6) NOT NULL,                    -- Item (6 dígitos)
    
    -- Características técnicas
    fabric_type VARCHAR(50),                          -- Tipo de tecido
    fabric_composition VARCHAR(100),                  -- Composição (% algodão, poliéster...)
    fabric_weight DECIMAL(6,2),                       -- Gramatura (g/m²)
    fabric_width DECIMAL(6,2),                        -- Largura (cm)
    color_type VARCHAR(20),                           -- liso, estampado, listrado, etc.
    
    -- Unidades de medida
    base_unit VARCHAR(10) DEFAULT 'metros',          -- metros, quilos, peças
    conversion_factor DECIMAL(10,4) DEFAULT 1.0,     -- Fator de conversão para base_unit
    
    -- Roteiro de produção (JSON estruturado)
    production_route JSONB NOT NULL DEFAULT '[]',
    /*
    Exemplo do JSON:
    [
      {
        "stage": "preparar",
        "stage_order": 1,
        "machine_type": "urdideira",
        "standard_time_min": 120,
        "next_stages": ["tingir"]
      },
      {
        "stage": "tingir",
        "stage_order": 2,
        "machine_type": "jigger",
        "standard_time_min": 300,
        "next_stages": ["secar"]
      }
    ]
    */
    
    -- Status
    is_active BOOLEAN DEFAULT TRUE,
    
    -- Auditoria
    created_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Índices
    INDEX idx_products_product_code (product_code),
    INDEX idx_products_group_code (group_code),
    INDEX idx_products_fabric_type (fabric_type),
    INDEX idx_products_is_active (is_active)
);

-- ============================================
-- TABELA: product_speeds (Velocidades por produto e etapa)
-- ============================================
CREATE TABLE product_speeds (
    id SERIAL PRIMARY KEY,
    product_code VARCHAR(20) REFERENCES products(product_code) ON DELETE CASCADE,
    production_stage VARCHAR(50) NOT NULL,            -- Etapa específica
    machine_type VARCHAR(50) NOT NULL,                -- Tipo de máquina
    
    -- Velocidades em metros por minuto
    standard_speed DECIMAL(10,2) NOT NULL,           -- Velocidade padrão
    min_speed DECIMAL(10,2),                         -- Velocidade mínima permitida
    max_speed DECIMAL(10,2),                         -- Velocidade máxima permitida
    
    -- Tempos auxiliares (minutos)
    setup_time_minutes INTEGER DEFAULT 30,           -- Tempo de setup
    cleaning_time_minutes INTEGER DEFAULT 15,        -- Tempo de limpeza
    changeover_time_minutes INTEGER DEFAULT 45,      -- Tempo de troca de produto
    
    -- Eficiências esperadas
    target_efficiency DECIMAL(5,2) DEFAULT 90.00,    -- Eficiência alvo (%)
    quality_standard DECIMAL(5,2) DEFAULT 98.00,     -- Padrão de qualidade (%)
    
    -- Fatores de ajuste
    color_factor_dark DECIMAL(5,2) DEFAULT 0.80,     -- Fator para cores escuras
    color_factor_light DECIMAL(5,2) DEFAULT 1.00,    -- Fator para cores claras
    complexity_factor_high DECIMAL(5,2) DEFAULT 0.70,-- Fator para estampas complexas
    
    -- Validação
    validated_by INTEGER REFERENCES users(id),
    validated_at TIMESTAMP,
    validation_notes TEXT,
    
    -- Auditoria
    created_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Restrições
    UNIQUE(product_code, production_stage, machine_type),
    CHECK (min_speed <= standard_speed),
    CHECK (max_speed >= standard_speed),
    
    -- Índices
    INDEX idx_product_speeds_product_code (product_code),
    INDEX idx_product_speeds_stage (production_stage),
    INDEX idx_product_speeds_machine_type (machine_type)
);

-- ============================================
-- TABELA: production_orders (Ordens de Produção do ERP)
-- ============================================
CREATE TABLE production_orders (
    id SERIAL PRIMARY KEY,
    op_number VARCHAR(50) UNIQUE NOT NULL,           -- Número da OP
    erp_id VARCHAR(50),                              -- ID no ERP (para sincronização)
    
    -- Produto
    product_code VARCHAR(20) REFERENCES products(product_code),
    product_description VARCHAR(200),                -- Cópia para performance
    
    -- Quantidades
    programmed_quantity DECIMAL(10,2) NOT NULL,      -- Quantidade programada
    programmed_unit VARCHAR(10) DEFAULT 'metros',    -- Unidade da programação
    loaded_quantity DECIMAL(10,2) NOT NULL,          -- Quantidade carregada
    produced_quantity DECIMAL(10,2) DEFAULT 0,       -- Quantidade produzida (acumulado)
    remaining_quantity DECIMAL(10,2) GENERATED ALWAYS AS (loaded_quantity - produced_quantity) STORED,
    
    -- Datas
    start_date DATE,                                 -- Data de início planejada
    end_date DATE,                                   -- Data de término planejada
    release_date TIMESTAMP,                          -- Data de liberação para produção
    completion_date TIMESTAMP,                       -- Data de conclusão real
    
    -- Prioridade
    priority INTEGER DEFAULT 1,                      -- 1 (baixa) a 5 (alta)
    customer_code VARCHAR(50),                       -- Código do cliente
    customer_name VARCHAR(100),                      -- Nome do cliente
    
    -- Status
    status VARCHAR(20) DEFAULT 'pending',            -- pending, released, in_progress, completed, cancelled
    current_stage VARCHAR(50),                       -- Estágio atual da produção
    next_stage VARCHAR(50),                          -- Próximo estágio
    
    -- Controle de qualidade
    quality_status VARCHAR(20) DEFAULT 'pending',    -- pending, approved, rejected, rework
    quality_notes TEXT,
    
    -- QR Code
    qr_code VARCHAR(255),                            -- Caminho do QR Code
    qr_code_data TEXT,                               -- Dados do QR Code
    
    -- Sincronização com ERP
    erp_sync_status VARCHAR(20) DEFAULT 'pending',   -- pending, synced, error
    erp_sync_date TIMESTAMP,
    erp_sync_notes TEXT,
    
    -- Auditoria
    created_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Índices
    INDEX idx_production_orders_op_number (op_number),
    INDEX idx_production_orders_status (status),
    INDEX idx_production_orders_product_code (product_code),
    INDEX idx_production_orders_dates (start_date, end_date),
    INDEX idx_production_orders_erp_sync (erp_sync_status)
);

-- ============================================
-- TABELA: machine_op_association (OPs em máquinas)
-- ============================================
CREATE TABLE machine_op_association (
    id SERIAL PRIMARY KEY,
    machine_id INTEGER REFERENCES machines(id) ON DELETE CASCADE,
    op_id INTEGER REFERENCES production_orders(id) ON DELETE CASCADE,
    
    -- Status da associação
    association_type VARCHAR(20) NOT NULL,           -- production, waiting, setup, maintenance, quality_check, material_wait
    association_status VARCHAR(20) DEFAULT 'active', -- active, paused, completed, cancelled
    
    -- Tempos
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    paused_at TIMESTAMP,                             -- Quando foi pausada
    resumed_at TIMESTAMP,                            -- Quando foi retomada
    total_pause_minutes INTEGER DEFAULT 0,           -- Tempo total pausado
    
    -- Responsáveis
    associated_by INTEGER REFERENCES users(id),      -- Quem associou
    operator_id INTEGER REFERENCES users(id),        -- Operador responsável
    
    -- Detalhes da produção
    expected_quantity DECIMAL(10,2),                 -- Quantidade esperada nesta máquina
    produced_quantity DECIMAL(10,2) DEFAULT 0,       -- Quantidade produzida
    
    -- Velocidade utilizada
    target_speed DECIMAL(10,2),                      -- Velocidade alvo (m/min)
    actual_speed DECIMAL(10,2),                      -- Velocidade real (calculada)
    
    -- Observações
    notes TEXT,
    
    -- Controle de concorrência
    is_active BOOLEAN DEFAULT TRUE,
    
    -- Auditoria
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Restrições
    UNIQUE(machine_id) WHERE is_active = TRUE,      -- Uma máquina só uma OP ativa
    CHECK (end_time IS NULL OR end_time > start_time),
    
    -- Índices
    INDEX idx_machine_op_machine (machine_id, is_active),
    INDEX idx_machine_op_op (op_id, is_active),
    INDEX idx_machine_op_type (association_type),
    INDEX idx_machine_op_times (start_time, end_time)
);

-- ============================================
-- TABELA: production_pointing (Apontamentos de produção)
-- ============================================
CREATE TABLE production_pointing (
    id SERIAL PRIMARY KEY,
    
    -- Referências
    user_id INTEGER REFERENCES users(id) NOT NULL,
    machine_id INTEGER REFERENCES machines(id) NOT NULL,
    op_id INTEGER REFERENCES production_orders(id) NOT NULL,
    association_id INTEGER REFERENCES machine_op_association(id),
    
    -- Período de produção
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    duration_minutes DECIMAL(8,2) GENERATED ALWAYS AS (
        EXTRACT(EPOCH FROM (end_time - start_time)) / 60
    ) STORED,
    
    -- Quantidades
    produced_quantity DECIMAL(10,2) NOT NULL,
    expected_quantity DECIMAL(10,2),                 -- Baseado na velocidade padrão
    good_quantity DECIMAL(10,2),                     -- Quantidade boa (após qualidade)
    rejected_quantity DECIMAL(10,2) DEFAULT 0,       -- Quantidade rejeitada
    
    -- Cálculos de eficiência
    standard_speed DECIMAL(10,2),                    -- Velocidade padrão do produto
    actual_speed DECIMAL(10,2),                      -- Velocidade real alcançada
    efficiency DECIMAL(5,2),                         -- Eficiência calculada
    performance DECIMAL(5,2),                        -- Desempenho (speed ratio)
    
    -- Detalhes
    production_stage VARCHAR(50) NOT NULL,           -- Estágio de produção
    shift VARCHAR(20),                               -- Turno
    notes TEXT,                                      -- Observações do operador
    
    -- Validação
    validated_by INTEGER REFERENCES users(id),
    validated_at TIMESTAMP,
    validation_status VARCHAR(20) DEFAULT 'pending', -- pending, approved, rejected
    
    -- Status
    pointing_status VARCHAR(20) DEFAULT 'completed', -- completed, incomplete, cancelled
    sync_status VARCHAR(20) DEFAULT 'pending',       -- pending, synced, error (para ERP)
    
    -- Auditoria
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Restrições
    CHECK (end_time > start_time),
    CHECK (produced_quantity >= 0),
    CHECK (good_quantity <= produced_quantity),
    
    -- Índices
    INDEX idx_pointing_user (user_id, start_time),
    INDEX idx_pointing_machine (machine_id, start_time),
    INDEX idx_pointing_op (op_id),
    INDEX idx_pointing_dates (start_time, end_time),
    INDEX idx_pointing_validation (validation_status)
);

-- ============================================
-- TABELA: stop_reasons (Motivos de parada)
-- ============================================
CREATE TABLE stop_reasons (
    id SERIAL PRIMARY KEY,
    code VARCHAR(20) UNIQUE NOT NULL,               -- Código interno
    description VARCHAR(100) NOT NULL,               -- Descrição
    category VARCHAR(50) NOT NULL,                   -- maintenance, material, quality, operational, external
    
    -- Classificação
    is_planned BOOLEAN DEFAULT FALSE,               -- Parada planejada?
    requires_approval BOOLEAN DEFAULT FALSE,        -- Requer aprovação?
    approval_role VARCHAR(50),                       -- Perfil que pode aprovar
    
    -- Impacto
    priority INTEGER DEFAULT 3,                      -- 1 (crítica) a 5 (baixa)
    expected_duration_minutes INTEGER,               -- Duração esperada
    affects_kpi BOOLEAN DEFAULT TRUE,               -- Afeta cálculo de OEE?
    
    -- Controle
    is_active BOOLEAN DEFAULT TRUE,
    
    -- Auditoria
    created_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Índices
    INDEX idx_stop_reasons_code (code),
    INDEX idx_stop_reasons_category (category),
    INDEX idx_stop_reasons_is_planned (is_planned)
);

-- ============================================
-- TABELA: machine_stops (Registro de paradas)
-- ============================================
CREATE TABLE machine_stops (
    id SERIAL PRIMARY KEY,
    
    -- Referências
    machine_id INTEGER REFERENCES machines(id) NOT NULL,
    stop_reason_id INTEGER REFERENCES stop_reasons(id) NOT NULL,
    reported_by INTEGER REFERENCES users(id) NOT NULL,
    op_id INTEGER REFERENCES production_orders(id),  -- OP afetada (se houver)
    
    -- Período da parada
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    duration_minutes DECIMAL(8,2) GENERATED ALWAYS AS (
        CASE 
            WHEN end_time IS NOT NULL THEN EXTRACT(EPOCH FROM (end_time - start_time)) / 60
            ELSE EXTRACT(EPOCH FROM (CURRENT_TIMESTAMP - start_time)) / 60
        END
    ) STORED,
    
    -- Detalhes
    notes TEXT,                                      -- Descrição detalhada
    action_taken TEXT,                               -- Ação tomada
    parts_used TEXT,                                 -- Peças/consumíveis utilizados
    
    -- Fotos/evidências
    photo_paths TEXT[],                              -- Caminhos das fotos
    
    -- Aprovação (se necessário)
    approved_by INTEGER REFERENCES users(id),
    approved_at TIMESTAMP,
    approval_notes TEXT,
    
    -- Status
    stop_status VARCHAR(20) DEFAULT 'active',        -- active, resolved, cancelled
    resolution_type VARCHAR(20),                     -- solved, workaround, pending
    
    -- Impacto na produção
    production_loss_quantity DECIMAL(10,2),          -- Quantidade perdida
    cost_impact DECIMAL(10,2),                       -- Impacto financeiro estimado
    
    -- Auditoria
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Restrições
    CHECK (end_time IS NULL OR end_time > start_time),
    
    -- Índices
    INDEX idx_machine_stops_machine (machine_id, start_time),
    INDEX idx_machine_stops_reason (stop_reason_id),
    INDEX idx_machine_stops_status (stop_status),
    INDEX idx_machine_stops_dates (start_time, end_time)
);

-- ============================================
-- TABELA: production_kpis (KPIs calculados)
-- ============================================
CREATE TABLE production_kpis (
    id SERIAL PRIMARY KEY,
    
    -- Dimensões
    kpi_date DATE NOT NULL,                          -- Data do KPI
    machine_id INTEGER REFERENCES machines(id),
    user_id INTEGER REFERENCES users(id),
    op_id INTEGER REFERENCES production_orders(id),
    department VARCHAR(50),
    shift VARCHAR(20),
    
    -- Métricas de tempo
    available_time_minutes INTEGER,                  -- Tempo disponível
    production_time_minutes INTEGER,                 -- Tempo em produção
    stop_time_minutes INTEGER,                       -- Tempo parado
    setup_time_minutes INTEGER,                      -- Tempo de setup
    cleaning_time_minutes INTEGER,                   -- Tempo de limpeza
    
    -- Métricas de quantidade
    produced_quantity DECIMAL(10,2),                 -- Quantidade produzida
    good_quantity DECIMAL(10,2),                     -- Quantidade boa
    rejected_quantity DECIMAL(10,2),                 -- Quantidade rejeitada
    expected_quantity DECIMAL(10,2),                 -- Quantidade esperada
    
    -- Cálculos de eficiência
    availability DECIMAL(5,2),                       -- Disponibilidade (%)
    performance DECIMAL(5,2),                        -- Desempenho (%)
    quality_rate DECIMAL(5,2),                       -- Taxa de qualidade (%)
    oee DECIMAL(5,2),                                -- OEE (%)
    
    -- Métricas de velocidade
    standard_speed_avg DECIMAL(10,2),                -- Velocidade padrão média
    actual_speed_avg DECIMAL(10,2),                  -- Velocidade real média
    speed_ratio DECIMAL(5,2),                        -- Relação velocidade
    
    -- Metas
    target_oee DECIMAL(5,2),                         -- Meta de OEE
    target_efficiency DECIMAL(5,2),                  -- Meta de eficiência
    variance DECIMAL(5,2),                           -- Variância em relação à meta
    
    -- Detalhes
    notes TEXT,
    
    -- Status do cálculo
    calculation_status VARCHAR(20) DEFAULT 'calculated', -- calculated, pending, error
    calculated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Auditoria
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Restrições
    UNIQUE(kpi_date, machine_id, shift),
    CHECK (available_time_minutes >= production_time_minutes),
    
    -- Índices
    INDEX idx_kpis_date (kpi_date),
    INDEX idx_kpis_machine (machine_id, kpi_date),
    INDEX idx_kpis_oee (oee),
    INDEX idx_kpis_department (department, kpi_date)
);

-- ============================================
-- TABELA: sync_logs (Logs de sincronização com ERP)
-- ============================================
CREATE TABLE sync_logs (
    id SERIAL PRIMARY KEY,
    sync_type VARCHAR(50) NOT NULL,                  -- orders, products, pointing, etc.
    direction VARCHAR(10) NOT NULL,                  -- import, export
    status VARCHAR(20) NOT NULL,                     -- success, error, partial
    
    -- Dados da sincronização
    records_processed INTEGER DEFAULT 0,
    records_successful INTEGER DEFAULT 0,
    records_failed INTEGER DEFAULT 0,
    
    -- Detalhes do erro (se houver)
    error_message TEXT,
    error_details JSONB,
    
    -- Tempos
    started_at TIMESTAMP NOT NULL,
    finished_at TIMESTAMP,
    duration_seconds DECIMAL(8,2),
    
    -- Configuração
    sync_config JSONB,                               -- Configuração usada
    
    -- Auditoria
    created_by INTEGER REFERENCES users(id),
    
    -- Índices
    INDEX idx_sync_logs_type (sync_type, started_at),
    INDEX idx_sync_logs_status (status),
    INDEX idx_sync_logs_dates (started_at)
);

-- ============================================
-- TABELA: audit_logs (Logs de auditoria)
-- ============================================
CREATE TABLE audit_logs (
    id SERIAL PRIMARY KEY,
    
    -- Informações básicas
    table_name VARCHAR(50) NOT NULL,                 -- Tabela afetada
    record_id INTEGER,                               -- ID do registro
    action VARCHAR(20) NOT NULL,                     -- INSERT, UPDATE, DELETE
    
    -- Usuário e contexto
    user_id INTEGER REFERENCES users(id),
    username VARCHAR(50),
    ip_address INET,
    user_agent TEXT,
    
    -- Dados alterados
    old_data JSONB,                                  -- Dados antigos
    new_data JSONB,                                  -- Dados novos
    changed_fields TEXT[],                           -- Campos alterados
    
    -- Detalhes
    description TEXT,
    
    -- Timestamp
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Índices
    INDEX idx_audit_table (table_name, created_at),
    INDEX idx_audit_user (user_id, created_at),
    INDEX idx_audit_action (action, created_at)
);

-- ============================================
-- CRIAR FUNÇÕES E TRIGGERS
-- ============================================

-- Função para atualizar updated_at automaticamente
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Aplicar trigger a todas as tabelas principais
DO $$
DECLARE
    tbl text;
BEGIN
    FOR tbl IN 
        SELECT table_name 
        FROM information_schema.tables 
        WHERE table_schema = 'public' 
        AND table_type = 'BASE TABLE'
        AND table_name NOT IN ('spatial_ref_sys', 'audit_logs')
    LOOP
        EXECUTE format('
            DROP TRIGGER IF EXISTS update_%s_updated_at ON %I;
            CREATE TRIGGER update_%s_updated_at
            BEFORE UPDATE ON %I
            FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
        ', tbl, tbl, tbl, tbl);
    END LOOP;
END$$;

-- Função para log de auditoria automático
CREATE OR REPLACE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $$
DECLARE
    old_data JSONB;
    new_data JSONB;
    changed_fields TEXT[];
    field_name TEXT;
BEGIN
    IF TG_OP = 'INSERT' THEN
        new_data = to_jsonb(NEW);
        old_data = '{}'::JSONB;
    ELSIF TG_OP = 'UPDATE' THEN
        new_data = to_jsonb(NEW);
        old_data = to_jsonb(OLD);
        
        -- Identificar campos alterados
        SELECT array_agg(key) INTO changed_fields
        FROM (
            SELECT key
            FROM jsonb_each(old_data)
            WHERE (old_data->key) IS DISTINCT FROM (new_data->key)
            UNION
            SELECT key
            FROM jsonb_each(new_data)
            WHERE (old_data->key) IS DISTINCT FROM (new_data->key)
        ) AS changed_keys;
    ELSIF TG_OP = 'DELETE' THEN
        new_data = '{}'::JSONB;
        old_data = to_jsonb(OLD);
    END IF;
    
    INSERT INTO audit_logs (
        table_name,
        record_id,
        action,
        old_data,
        new_data,
        changed_fields,
        description
    ) VALUES (
        TG_TABLE_NAME,
        COALESCE(NEW.id, OLD.id),
        TG_OP,
        old_data,
        new_data,
        changed_fields,
        TG_TABLE_NAME || ' ' || TG_OP || ' operation'
    );
    
    IF TG_OP = 'DELETE' THEN
        RETURN OLD;
    ELSE
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Aplicar triggers de auditoria a tabelas críticas
CREATE TRIGGER audit_users
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

CREATE TRIGGER audit_production_orders
AFTER INSERT OR UPDATE OR DELETE ON production_orders
FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

CREATE TRIGGER audit_production_pointing
AFTER INSERT OR UPDATE OR DELETE ON production_pointing
FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

CREATE TRIGGER audit_machine_stops
AFTER INSERT OR UPDATE OR DELETE ON machine_stops
FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

-- Função para calcular eficiência automática
CREATE OR REPLACE FUNCTION calculate_efficiency(
    p_produced_quantity DECIMAL,
    p_expected_quantity DECIMAL,
    p_standard_speed DECIMAL,
    p_actual_speed DECIMAL
)
RETURNS DECIMAL AS $$
BEGIN
    IF p_expected_quantity > 0 THEN
        RETURN (p_produced_quantity / p_expected_quantity) * 100;
    ELSIF p_standard_speed > 0 AND p_actual_speed > 0 THEN
        RETURN (p_actual_speed / p_standard_speed) * 100;
    ELSE
        RETURN 0;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- ============================================
-- INSERIR DADOS INICIAIS (SEED)
-- ============================================

-- Inserir usuário admin padrão
-- Senha: Admin@2024 (será criptografada na aplicação)
INSERT INTO users (employee_code, username, password_hash, name, user_type, is_active) 
VALUES 
('000001', 'admin', '$argon2id$v=19$m=65536,t=3,p=4$c2FsdHNhbHQ$O5j6Z4YJwQjRkZgKtL3vLmZ4YJwQjRkZgKtL3vLm', 'Administrador do Sistema', 'admin', true);

-- Inserir motivos de parada padrão
INSERT INTO stop_reasons (code, description, category, is_planned, priority) VALUES
('MAN-001', 'Manutenção Corretiva', 'maintenance', false, 1),
('MAN-002', 'Manutenção Preventiva', 'maintenance', true, 3),
('MAT-001', 'Falta de Matéria-Prima', 'material', false, 1),
('MAT-002', 'Aguardando OP Anterior', 'material', false, 2),
('OPE-001', 'Troca de Turno', 'operational', true, 4),
('OPE-002', 'Reunião/Treinamento', 'operational', true, 4),
('OPE-003', 'Almoço/Intervalo', 'operational', true, 5),
('QUA-001', 'Ajuste de Qualidade', 'quality', false, 2),
('QUA-002', 'Aguardando Liberação', 'quality', false, 2),
('EXT-001', 'Falta de Energia', 'external', false, 1),
('EXT-002', 'Falta de Água/Gás', 'external', false, 1),
('EXT-003', 'Falta de Ar Comprimido', 'external', false, 1);

-- Inserir tipos de máquina padrão
INSERT INTO machine_types (code, name, description) VALUES
('URD', 'Urdideira', 'Preparação dos fios para tecelagem'),
('ENR', 'Enroladeira', 'Enrolamento dos tecidos'),
('REV', 'Revisadeira', 'Inspeção e revisão de tecidos'),
('JIG', 'Jigger', 'Tingimento descontínuo'),
('TUR', 'Turbo', 'Tingimento rápido'),
('STO', 'Stork', 'Estamparia de tecidos'),
('RAM', 'Rama', 'Tingimento contínuo'),
('CAL', 'Calandra', 'Acabamento com calor e pressão'),
('SAN', 'Sanforizadeira', 'Estabilização dimensional'),
('SEC', 'Secadeira', 'Secagem de tecidos');

COMMIT;

-- ============================================
-- CRIAR USUÁRIOS E PERMISSÕES DO BANCO
-- ============================================

-- Conceder permissões ao usuário da aplicação
GRANT CONNECT ON DATABASE production_pointer TO production_user;
GRANT USAGE ON SCHEMA public TO production_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO production_user;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO production_user;
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO production_user;

-- Criar usuário para backups
CREATE USER backup_user WITH PASSWORD 'B@ckup2024!';
GRANT CONNECT ON DATABASE production_pointer TO backup_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO backup_user;

-- Criar usuário para relatórios (somente leitura)
CREATE USER report_user WITH PASSWORD 'R3port2024!';
GRANT CONNECT ON DATABASE production_pointer TO report_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO report_user;

-- Configurar políticas de segurança (se usando PostgreSQL 9.5+)
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
CREATE POLICY users_policy ON users FOR ALL TO production_user USING (true);

-- ============================================
-- CRIAR VIEWS PARA RELATÓRIOS
-- ============================================

-- View para dashboard de produção
CREATE VIEW vw_production_dashboard AS
SELECT 
    DATE(pp.start_time) as production_date,
    m.department,
    m.machine_type,
    COUNT(DISTINCT pp.op_id) as ops_count,
    SUM(pp.produced_quantity) as total_produced,
    SUM(pp.expected_quantity) as total_expected,
    AVG(pp.efficiency) as avg_efficiency,
    COUNT(ms.id) as stops_count,
    SUM(ms.duration_minutes) as total_stop_time
FROM production_pointing pp
JOIN machines m ON pp.machine_id = m.id
LEFT JOIN machine_stops ms ON pp.machine_id = ms.machine_id 
    AND ms.start_time::date = pp.start_time::date
    AND ms.stop_status = 'resolved'
GROUP BY DATE(pp.start_time), m.department, m.machine_type;

-- View para análise OEE
CREATE VIEW vw_oee_analysis AS
SELECT 
    pk.kpi_date,
    m.code as machine_code,
    m.name as machine_name,
    m.department,
    pk.availability,
    pk.performance,
    pk.quality_rate,
    pk.oee,
    pk.target_oee,
    (pk.oee - pk.target_oee) as variance
FROM production_kpis pk
JOIN machines m ON pk.machine_id = m.id
WHERE pk.calculation_status = 'calculated';

-- View para tempo de parada por motivo
CREATE VIEW vw_stop_analysis AS
SELECT 
    DATE(ms.start_time) as stop_date,
    sr.category,
    sr.description as stop_reason,
    COUNT(*) as occurrences,
    SUM(ms.duration_minutes) as total_minutes,
    AVG(ms.duration_minutes) as avg_minutes,
    STRING_AGG(DISTINCT m.code, ', ') as affected_machines
FROM machine_stops ms
JOIN stop_reasons sr ON ms.stop_reason_id = sr.id
JOIN machines m ON ms.machine_id = m.id
WHERE ms.stop_status = 'resolved'
GROUP BY DATE(ms.start_time), sr.category, sr.description;

-- ============================================
-- CONFIGURAR BACKUP AUTOMÁTICO
-- ============================================

-- Criar função para backup lógico
CREATE OR REPLACE FUNCTION backup_production_data()
RETURNS void AS $$
BEGIN
    -- Esta função será chamada por scripts externos
    -- O backup real é feito via pg_dump no sistema operacional
    RAISE NOTICE 'Backup function called at %', NOW();
END;
$$ LANGUAGE plpgsql;

-- Configurar replicação lógica (opcional, para alta disponibilidade)
-- ALTER SYSTEM SET wal_level = logical;
-- ALTER SYSTEM SET max_replication_slots = 10;
-- ALTER SYSTEM SET max_wal_senders = 10;

SELECT 'Banco de dados Production Pointer criado com sucesso!' as message;
```

### **6.2 Modelo Físico Otimizado para Performance**

#### **Particionamento de Tabelas Grandes**
```sql
-- Particionar production_pointing por mês
CREATE TABLE production_pointing_y2024m01 PARTITION OF production_pointing
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE production_pointing_y2024m02 PARTITION OF production_pointing
FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
-- Continuar para outros meses...

-- Particionar machine_stops por trimestre
CREATE TABLE machine_stops_y2024q1 PARTITION OF machine_stops
FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');
```

#### **Índices para Performance**
```sql
-- Índices compostos para queries frequentes
CREATE INDEX idx_pointing_machine_date ON production_pointing(machine_id, start_time DESC);
CREATE INDEX idx_pointing_op_status ON production_pointing(op_id, pointing_status);
CREATE INDEX idx_stops_machine_date ON machine_stops(machine_id, start_time DESC);
CREATE INDEX idx_orders_status_date ON production_orders(status, start_date);
CREATE INDEX idx_association_active ON machine_op_association(is_active, machine_id);
```

#### **Materialized Views para Relatórios**
```sql
-- View materializada para dashboard em tempo real
CREATE MATERIALIZED VIEW mv_dashboard_realtime AS
SELECT 
    m.id as machine_id,
    m.code as machine_code,
    m.name as machine_name,
    m.status as machine_status,
    COALESCE(moa.association_type, 'idle') as current_status,
    po.op_number,
    po.product_description,
    EXTRACT(EPOCH FROM (NOW() - moa.start_time)) / 60 as current_duration_minutes,
    u.name as operator_name
FROM machines m
LEFT JOIN machine_op_association moa ON m.id = moa.machine_id AND moa.is_active = true
LEFT JOIN production_orders po ON moa.op_id = po.id
LEFT JOIN users u ON moa.operator_id = u.id
WHERE m.is_active = true;

-- Atualizar a cada 5 minutos
CREATE OR REPLACE FUNCTION refresh_dashboard_view()
RETURNS trigger AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY mv_dashboard_realtime;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```

---

## **7. FLUXOS DE TRABALHO DETALHADOS**

### **7.1 Fluxo Completo de Apontamento de Produção**

#### **7.1.1 Diagrama de Estados do Sistema**
```
┌─────────────────────────────────────────────────────────────┐
│                 ESTADOS DO SISTEMA DE APONTAMENTO           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────┐     ┌─────────┐     ┌──────────┐     ┌──────┐│
│  │   OCIO  │────▶│ SETUP   │────▶│PRODUZINDO│────▶│PARADA││
│  └─────────┘     └─────────┘     └──────────┘     └──────┘│
│       │               │               │               │    │
│       │               │               │               │    │
│  ┌────▼───────────────▼───────────────▼───────────────▼────┐│
│  │                    QUALIDADE                            ││
│  │                 (Controle/Inspeção)                     ││
│  └─────────────────────────────────────────────────────────┘│
│                                                             │
│  Estados Secundários:                                       │
│  • Aguardando Material                                      │
│  • Manutenção                                               │
│  • Limpeza                                                  │
│  • Calibração                                               │
│  • Esperando Liberação                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### **7.1.2 Fluxo Principal: Operador no Chão de Fábrica**
```
┌─────────────────────────────────────────────────────────────┐
│                 FLUXO PRINCIPAL - OPERADOR                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. LOGIN NO SISTEMA                                        │
│     • Usuário/senha ou biometria                           │
│     • Sistema valida turno e permissões                    │
│     • Carrega máquinas atribuídas ao operador              │
│                                                             │
│  2. SELEÇÃO/SCAN DE MÁQUINA                                │
│     ┌─────────────────────────────────────┐                │
│     │ Opção 1: Lista de máquinas          │                │
│     │ Opção 2: Scan QR Code da máquina    │                │
│     └─────────────────────────────────────┘                │
│                                                             │
│  3. VERIFICAÇÃO DO STATUS DA MÁQUINA                       │
│     ├─── CASO 1: Máquina LIVRE (sem OP) ─────────────┐    │
│     │     • Sistema sugere OPs disponíveis           │    │
│     │     • Operador seleciona OP                    │    │
│     │     └─▶ Vá para passo 4                        │    │
│     │                                                │    │
│     ├─── CASO 2: Máquina com OP em PRODUÇÃO ────────┐│    │
│     │     • Mostra OP em produção                   ││    │
│     │     • Oferece opções:                         ││    │
│     │       - Continuar produção                    ││    │
│     │       - Registrar parada                      ││    │
│     │       - Adicionar observação                  ││    │
│     │       - Finalizar produção                    ││    │
│     │     └─▶ Vá para passo 5 ou 6                  ││    │
│     │                                                ││    │
│     ├─── CASO 3: Máquina PARADA com OP ─────────────┐││    │
│     │     • Mostra OP parada e motivo               │││    │
│     │     • Oferece opções:                         │││    │
│     │       - Retomar produção                      │││    │
│     │       - Registrar novo motivo                 │││    │
│     │       - Transferir OP                         │││    │
│     │       - Finalizar OP                          │││    │
│     │     └─▶ Vá para passo 5 ou 7                  │││    │
│     │                                                │││    │
│     └─── CASO 4: Máquina em MANUTENÇÃO ─────────────┐│││    │
│           • Mostra detalhes da manutenção           ││││    │
│           • Oferece opções:                         ││││    │
│             - Registrar conclusão                   ││││    │
│             - Adicionar observação                  ││││    │
│             - Continuar manutenção                  ││││    │
│           └─▶ Vá para passo 8                       ││││    │
│                                                    │││││    │
│  4. INICIAR NOVA PRODUÇÃO (Máquina livre)          │││││    │
│     • Scan QR Code da OP                           │││││    │
│     • Sistema valida:                              │││││    │
│       - OP está liberada?                          │││││    │
│       - Máquina compatível?                        │││││    │
│       - Etapa correta do roteiro?                  │││││    │
│     • Confirma dados:                              │││││    │
│       - Metragem programada                        │││││    │
│       - Velocidade padrão do produto               │││││    │
│       - Tempo estimado                             │││││    │
│     • Operador confirma                            │││││    │
│     • Sistema inicia cronômetro                    │││││    │
│     └─▶ Vá para passo 5                            │││││    │
│                                                    │││││    │
│  5. PRODUÇÃO EM ANDAMENTO                          │││││    │
│     • Tela mostra:                                 │││││    │
│       - Cronômetro de produção                     │││││    │
│       - Metragem produzida (estimada)              │││││    │
│       - Eficiência em tempo real                   │││││    │
│       - Velocidade atual                           │││││    │
│     • Botões disponíveis:                          │││││    │
│       - ⏸️  Registrar Parada                      │││││    │
│       - 📝 Adicionar Observação                   │││││    │
│       - ✅ Finalizar Produção                     │││││    │
│       - 🔄 Trocar Operador (se supervisor)         │││││    │
│                                                    │││││    │
│  6. REGISTRAR PARADA DURANTE PRODUÇÃO              │││││    │
│     • Sistema para cronômetro de produção          │││││    │
│     • Mostra categorias de parada:                 │││││    │
│       - Manutenção                                 │││││    │
│       - Material                                   │││││    │
│       - Qualidade                                  │││││    │
│       - Operacional                                │││││    │
│       - Externa                                    │││││    │
│     • Operador seleciona motivo específico         │││││    │
│     • Adiciona observações (opcional)              │││││    │
│     • Tira foto (opcional)                         │││││    │
│     • Sistema inicia cronômetro de parada          │││││    │
│     • Notifica supervisor se parada > 30min        │││││    │
│                                                    │││││    │
│  7. FINALIZAR PRODUÇÃO                             │││││    │
│     • Operador escaneia QR Code da OP novamente    │││││    │
│     • Sistema mostra:                              │││││    │
│       - Metragem produzida (calculada)             │││││    │
│       - Tempo total de produção                    │││││    │
│       - Eficiência alcançada                       │││││    │
│       - Comparação com meta                        │││││    │
│     • Operador pode:                               │││││    │
│       - ✅ Confirmar metragem                      │││││    │
│       - ✏️  Ajustar metragem (com justificativa)  │││││    │
│       - 📝 Adicionar observações                  │││││    │
│       - 🏷️  Registrar problemas de qualidade      │││││    │
│     • Sistema:                                     │││││    │
│       - Calcula KPIs finais                        │││││    │
│       - Atualiza status da OP                      │││││    │
│       - Libera máquina                             │││││    │
│       - Envia notificação ao supervisor            │││││    │
│       - Sincroniza com ERP (se configurado)        │││││    │
│                                                    │││││    │
│  8. REGISTRAR CONCLUSÃO DE MANUTENÇÃO              │││││    │
│     • Operador seleciona "Concluir Manutenção"     │││││    │
│     • Sistema solicita:                            │││││    │
│       - Peças/substituídos utilizados              │││││    │
│       - Tempo gasto                                │││││    │
│       - Observações                                │││││    │
│       - Fotos (antes/depois)                       │││││    │
│     • Supervisor aprova (se necessário)            │││││    │
│     • Sistema atualiza histórico da máquina        │││││    │
│     • Agenda próxima manutenção preventiva         │││││    │
│                                                    │││││    │
│  9. TROCA DE TURNO                                 │││││    │
│     • Sistema detecta fim de turno                 │││││    │
│     • Solicita ao operador:                        │││││    │
│       - Finalizar produções em andamento           │││││    │
│       - Registrar parada de troca de turno         │││││    │
│       - Passar informações para próximo turno      │││││    │
│     • Gera relatório de turno                      │││││    │
│                                                    │││││    │
└─────────────────────────────────────────────────────┘││││    │
                                                      │││││    │
┌─────────────────────────────────────────────────────┐││││    │
│                FLUXOS ESPECIAIS                     │││││    │
├─────────────────────────────────────────────────────┤││││    │
│                                                    │││││    │
│  A. TRANSFERÊNCIA DE OP ENTRE MÁQUINAS             │││││    │
│     • Motivos:                                     │││││    │
│       - Quebra de máquina                          │││││    │
│       - Otimização de capacidade                   │││││    │
│       - Priorização de OP urgente                  │││││    │
│     • Processo:                                    │││││    │
│       1. Finalizar OP na máquina origem            │││││    │
│       2. Registrar motivo da transferência         │││││    │
│       3. Escanear máquina destino                  │││││    │
│       4. Sistema verifica compatibilidade          │││││    │
│       5. Iniciar produção na máquina destino       │││││    │
│       6. Registrar tempo de setup adicional        │││││    │
│                                                    │││││    │
│  B. REPROCESSAMENTO/REWORK                         │││││    │
│     • Quando: Qualidade rejeita lote               │││││    │
│     • Processo:                                    │││││    │
│       1. Qualidade registra não conformidade       │││││    │
│       2. Sistema bloqueia OP para produção         │││││    │
│       3. Supervisor/autorizado libera reprocesso   │││││    │
│       4. OP volta para etapa específica            │││││    │
│       5. Produção registra como "rework"           │││││    │
│       6. Custo adicional é calculado               │││││    │
│                                                    │││││    │
│  C. PRODUÇÃO EM LOTE PEQUENO (AMOSTRAS)           │││││    │
│     • Para: Desenvolvimento, testes, clientes      │││││    │
│     • Processo:                                    │││││    │
│       1. Criar OP especial (tipo "amostra")        │││││    │
│       2. Velocidades podem ser diferentes          │││││    │
│       3. Registro detalhado de parâmetros          │││││    │
│       4. Não conta para eficiência padrão          │││││    │
│                                                    │││││    │
│  D. PRODUÇÃO COM MATERIAIS ALTERNATIVOS            │││││    │
│     • Quando: Falta material padrão                │││││    │
│     • Processo:                                    │││││    │
│       1. Supervisor autoriza substituição          │││││    │
│       2. Sistema registra material alternativo     │││││    │
│       3. Ajusta velocidades esperadas              │││││    │
│       4. Notifica qualidade sobre alteração        │││││    │
│                                                    │││││    │
└─────────────────────────────────────────────────────┘││││    │
                                                      │││││    │
┌─────────────────────────────────────────────────────┐││││    │
│                CONTROLE DE QUALIDADE                │││││    │
├─────────────────────────────────────────────────────┤││││    │
│                                                    │││││    │
│  1. INSPEÇÃO DURANTE PRODUÇÃO                      │││││    │
│     • Sistema gera alertas para amostragem         │││││    │
│     • Inspetor escaneia OP em produção             │││││    │
│     • Registra:                                    │││││    │
│       - Dimensões                                  │││││    │
│       - Cor                                        │││││    │
│       - Defeitos                                   │││││    │
│       - Fotos                                      │││││    │
│     • Sistema calcula:                             │││││    │
│       - Taxa de defeito                            │││││    │
│       - Tendência de qualidade                     │││││    │
│                                                    │││││    │
│  2. BLOQUEIO/LIBERAÇÃO DE LOTE                     │││││    │
│     • Se qualidade abaixo do padrão:               │││││    │
│       - Sistema bloqueia OP                        │││││    │
│       - Notifica produção e planejamento           │││││    │
│       - Sugere ações corretivas                    │││││    │
│     • Se qualidade aprovada:                       │││││    │
│       - Sistema libera para próxima etapa          │││││    │
│       - Atualiza status da OP                      │││││    │
│                                                    │││││    │
│  3. ANÁLISE DE CAUSA-RAIZ                          │││││    │
│     • Sistema agrupa defeitos por:                 │││││    │
│       - Máquina                                    │││││    │
│       - Operador                                   │││││    │
│       - Turno                                      │││││    │
│       - Material                                   │││││    │
│     • Gera relatórios para ações preventivas       │││││    │
│                                                    │││││    │
└─────────────────────────────────────────────────────┘││││    │
                                                      │││││    │
└──────────────────────────────────────────────────────┘│││    │
                                                       ││││    │
└───────────────────────────────────────────────────────┘││    │
                                                        │││    │
└────────────────────────────────────────────────────────┘│    │
                                                         ││    │
└─────────────────────────────────────────────────────────┘    │
                                                              │
┌──────────────────────────────────────────────────────────────┐
│                 FLUXO DE DADOS - BACKEND                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FRONTEND (Browser/App) → NGINX → GUNICORN → FLASK APP      │
│                                                              │
│  1. Autenticação JWT                                        │
│  2. Validação de permissões                                 │
│  3. Processamento da requisição                             │
│  4. Interação com banco de dados                            │
│  5. Cálculos em tempo real                                  │
│  6. Resposta ao frontend                                    │
│  7. Logs de auditoria                                       │
│  8. Sincronização com ERP (background job)                  │
│                                                              │
│  Componentes envolvidos:                                    │
│  • Flask-RESTful: API endpoints                             │
│  • SQLAlchemy: ORM para PostgreSQL                         │
│  • Redis: Cache e filas                                     │
│  • Celery: Tarefas assíncronas                              │
│  • SocketIO: Comunicação em tempo real                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### **7.2 Regras de Negócio Detalhadas**

#### **7.2.1 Regras de Validação de Produção**
```python
# Exemplo de regras de negócio em Python
class ProductionRules:
    
    @staticmethod
    def validate_op_for_machine(op_id, machine_id, user_id):
        """
        Validar se OP pode ser produzida em determinada máquina
        """
        # 1. Verificar se OP existe e está liberada
        op = ProductionOrder.query.get(op_id)
        if not op or op.status != 'released':
            return False, "OP não encontrada ou não liberada"
        
        # 2. Verificar se máquina existe e está disponível
        machine = Machine.query.get(machine_id)
        if not machine or machine.status not in ['stopped', 'idle']:
            return False, "Máquina não disponível"
        
        # 3. Verificar compatibilidade de etapa
        current_stage = op.current_stage
        required_machine_type = get_machine_type_for_stage(current_stage)
        
        if machine.machine_type != required_machine_type:
            return False, f"Máquina incompatível. Necessário: {required_machine_type}"
        
        # 4. Verificar se operador tem permissão
        user = User.query.get(user_id)
        allowed_machines = user.get_allowed_machines()
        
        if machine_id not in allowed_machines:
            return False, "Operador não autorizado para esta máquina"
        
        # 5. Verificar se há material disponível (integração com ERP)
        material_available = check_material_availability(op.product_code)
        if not material_available:
            return False, "Material insuficiente"
        
        # 6. Verificar prioridade vs outras OPs na fila
        higher_priority_ops = check_higher_priority_ops(machine.department)
        if higher_priority_ops:
            return False, f"Existem {len(higher_priority_ops)} OPs com prioridade maior"
        
        return True, "Validação aprovada"
    
    @staticmethod
    def calculate_expected_production_time(op_id, machine_id):
        """
        Calcular tempo esperado de produção baseado no produto e máquina
        """
        op = ProductionOrder.query.get(op_id)
        machine = Machine.query.get(machine_id)
        
        # Obter velocidade padrão do produto nesta etapa
        speed_info = ProductSpeed.query.filter_by(
            product_code=op.product_code,
            production_stage=op.current_stage,
            machine_type=machine.machine_type
        ).first()
        
        if not speed_info:
            # Usar velocidade padrão da máquina
            standard_speed = machine.standard_speed
        else:
            standard_speed = speed_info.standard_speed
        
        # Calcular tempo de produção
        production_time = op.remaining_quantity / standard_speed  # em minutos
        
        # Adicionar tempo de setup
        setup_time = speed_info.setup_time_minutes if speed_info else 30
        
        total_time = production_time + setup_time
        
        return {
            'production_minutes': production_time,
            'setup_minutes': setup_time,
            'total_minutes': total_time,
            'standard_speed': standard_speed,
            'expected_finish': datetime.now() + timedelta(minutes=total_time)
        }
    
    @staticmethod
    def validate_final_quantity(op_id, reported_quantity, user_id):
        """
        Validar quantidade final reportada
        """
        op = ProductionOrder.query.get(op_id)
        
        # 1. Não pode exceder quantidade carregada
        if reported_quantity > op.loaded_quantity:
            return False, "Quantidade reportada excede quantidade carregada"
        
        # 2. Verificar tolerância (configurável, padrão ±5%)
        tolerance = 0.05  # 5%
        expected_min = op.loaded_quantity * (1 - tolerance)
        expected_max = op.loaded_quantity * (1 + tolerance)
        
        if reported_quantity < expected_min:
            return False, f"Quantidade abaixo do mínimo esperado ({expected_min:.2f})"
        
        # 3. Se quantidade muito diferente, requer aprovação
        if reported_quantity < op.loaded_quantity * 0.9:  # Menos de 90%
            require_supervisor_approval = True
        else:
            require_supervisor_approval = False
        
        return True, "Validação aprovada", require_supervisor_approval
```

#### **7.2.2 Regras de Parada de Máquina**
```python
class StopRules:
    
    @staticmethod
    def classify_stop_duration(stop_reason_id, duration_minutes):
        """
        Classificar duração da parada e definir ações
        """
        reason = StopReason.query.get(stop_reason_id)
        
        if reason.category == 'maintenance':
            if duration_minutes > 120:  # Mais de 2 horas
                return 'critical', ['notify_maintenance_manager', 'escalate_to_production_manager']
            elif duration_minutes > 60:  # 1-2 horas
                return 'high', ['notify_maintenance_supervisor']
            else:
                return 'normal', []
        
        elif reason.category == 'material':
            if duration_minutes > 90:  # Mais de 1.5 horas
                return 'critical', ['notify_supply_chain', 'escalate_to_production_manager']
            elif duration_minutes > 30:  # 30-90 minutos
                return 'high', ['notify_warehouse']
            else:
                return 'normal', []
        
        elif reason.category == 'quality':
            if duration_minutes > 60:
                return 'high', ['notify_quality_manager']
            else:
                return 'normal', []
        
        else:
            return 'normal', []
    
    @staticmethod
    def calculate_production_loss(machine_id, stop_duration, op_id=None):
        """
        Calcular perda de produção devido à parada
        """
        machine = Machine.query.get(machine_id)
        
        if op_id:
            # Parada durante produção específica
            op = ProductionOrder.query.get(op_id)
            speed_info = ProductSpeed.query.filter_by(
                product_code=op.product_code,
                production_stage=op.current_stage,
                machine_type=machine.machine_type
            ).first()
            
            if speed_info:
                standard_speed = speed_info.standard_speed
            else:
                standard_speed = machine.standard_speed
        else:
            # Parada sem OP específica
            standard_speed = machine.standard_speed
        
        # Calcular metros perdidos
        lost_meters = standard_speed * stop_duration
        
        # Estimar custo (R$/metro × metros perdidos)
        cost_per_meter = get_cost_per_meter(machine.department)
        estimated_cost = lost_meters * cost_per_meter
        
        return {
            'lost_meters': lost_meters,
            'estimated_cost': estimated_cost,
            'standard_speed': standard_speed,
            'stop_duration': stop_duration
        }
```

### **7.3 Fluxos de Exceção e Contingência**

#### **7.3.1 Falha de Conexão com Servidor**
```
Quando: Dispositivo móvel perde conexão Wi-Fi

Procedimento:
1. Sistema detecta perda de conexão
2. Armazena dados localmente (localStorage/IndexedDB)
3. Mostra indicador "Modo Offline" para operador
4. Permite continuar apontamentos normalmente
5. Quando conexão restaurada:
   - Sincroniza dados pendentes
   - Valida consistência
   - Confirma sincronização com usuário
6. Se conflitos detectados:
   - Mostra diferenças
   - Permite escolher versão correta
   - Registra resolução para auditoria
```

#### **7.3.2 QR Code Danificado ou Ile gível**
```
Quando: QR Code raspado, sujo ou ilegível

Procedimento:
1. Operador seleciona "QR Code ilegível"
2. Sistema oferece alternativas:
   a) Digitar código manualmente
   b) Buscar por código/nome
   c) Usar leitor de código de barras
   d) Escolher da lista recente
3. Sistema registra o ocorrido para:
   - Gerar novo QR Code (se necessário)
   - Manutenção preventiva (substituição)
```

#### **7.3.3 Erro do Operador**
```
Quando: Operador comete erro no apontamento

Procedimento (dependendo do estágio):

CASO A: Erro antes de confirmar
1. Operador pode cancelar/corrigir livremente
2. Sistema mantém histórico de alterações

CASO B: Erro após confirmação (produção em andamento)
1. Operador solicita correção
2. Sistema exige justificativa
3. Supervisor aprova (se necessário)
4. Sistema registra correção com auditoria

CASO C: Erro após finalização
1. Operador não pode alterar diretamente
2. Deve solicitar ao supervisor
3. Supervisor analisa e autoriza correção
4. Sistema mantém ambas versões (original e corrigida)
```

#### **7.3.4 Problemas com Velocidade de Produção**
```
Quando: Velocidade real muito diferente da padrão

Procedimento:
1. Sistema detecta desvio > 20% da velocidade padrão
2. Alerta operador e supervisor
3. Oferece opções:
   a) Continuar com velocidade atual (registra motivo)
   b) Ajustar velocidade para padrão
   c) Parar para verificação técnica
4. Se escolher continuar:
   - Registra velocidade real
   - Solicita observação obrigatória
   - Ajusta expectativas de produção
5. Atualiza histórico para análise futura
```

---

## **8. INTEGRAÇÃO COM ERP SYSTÊXTIL**

### **8.1 Arquitetura de Integração**
```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│  ERP SYSTÊXTIL  │      │   PRODUCTION    │      │    DISPOSITIVOS │
│                 │      │    POINTER      │      │      MÓVEIS     │
├─────────────────┤      ├─────────────────┤      ├─────────────────┤
│ • API REST      │◄────▶│ • Sync Service  │◄────▶│ • App Web       │
│ • Banco Dados   │      │ • Cache Local   │      │ • PWA/Offline   │
│ • Business Logic│      │ • Business Rules│      │ • QR Scanner    │
└─────────────────┘      └─────────────────┘      └─────────────────┘
       │                         │                         │
       │                         │                         │
┌──────▼─────────────────────────▼─────────────────────────▼──────┐
│                    PROTOCOLO DE COMUNICAÇÃO                     │
│  • HTTPS com TLS 1.2+                                          │
│  • Autenticação: API Key + Token                               │
│  • Formato: JSON                                               │
│  • Encoding: UTF-8                                             │
└─────────────────────────────────────────────────────────────────┘
```

### **8.2 Endpoints da API do Systêxtil (Exemplo)**

#### **8.2.1 Obter Ordens de Produção**
```http
GET /api/v1/production-orders
Headers:
  Authorization: Bearer {api_token}
  X-API-Key: {api_key}
  
Query Parameters:
  status: released|in_progress|completed  (opcional)
  from_date: YYYY-MM-DD                  (opcional)
  to_date: YYYY-MM-DD                    (opcional)
  department: string                     (opcional)

Response:
{
  "success": true,
  "data": [
    {
      "op_number": "OP-2024-001234",
      "erp_id": "123456",
      "product_code": "1.23456.789.123456",
      "product_description": "Tecido Algodão 200g/m² Branco",
      "programmed_quantity": 5000.00,
      "loaded_quantity": 5000.00,
      "produced_quantity": 0.00,
      "unit": "metros",
      "start_date": "2024-01-15",
      "end_date": "2024-01-17",
      "priority": 1,
      "status": "released",
      "customer_code": "CLI-001",
      "customer_name": "Confecções Moda Ltda",
      "production_route": [
        {"stage": "preparar", "machine_type": "urdideira"},
        {"stage": "tingir", "machine_type": "jigger"},
        {"stage": "secar", "machine_type": "secadeira"}
      ],
      "created_at": "2024-01-14T08:00:00Z",
      "updated_at": "2024-01-14T08:00:00Z"
    }
  ],
  "pagination": {
    "total": 150,
    "page": 1,
    "per_page": 50,
    "total_pages": 3
  }
}
```

#### **8.2.2 Enviar Apontamentos para o ERP**
```http
POST /api/v1/production-pointing
Headers:
  Authorization: Bearer {api_token}
  Content-Type: application/json

Request Body:
{
  "pointing_data": [
    {
      "op_number": "OP-2024-001234",
      "machine_code": "JIG-01",
      "operator_code": "FUNC-001",
      "start_time": "2024-01-15T08:30:00-03:00",
      "end_time": "2024-01-15T12:45:00-03:00",
      "produced_quantity": 1250.50,
      "good_quantity": 1225.25,
      "rejected_quantity": 25.25,
      "rejection_reasons": [
        {"code": "MAN-001", "quantity": 15.00},
        {"code": "QUA-002", "quantity": 10.25}
      ],
      "production_stage": "tingir",
      "standard_speed": 15.00,
      "actual_speed": 14.80,
      "efficiency": 98.67,
      "notes": "Tecido com ótima qualidade, sem problemas",
      "stop_events": [
        {
          "stop_reason": "OPE-003",
          "start_time": "2024-01-15T10:00:00-03:00",
          "end_time": "2024-01-15T10:15:00-03:00",
          "duration_minutes": 15,
          "notes": "Intervalo café"
        }
      ],
      "quality_checks": [
        {
          "check_type": "color",
          "result": "approved",
          "value": "5.2",
          "unit": "deltaE",
          "notes": "Dentro da tolerância"
        }
      ]
    }
  ]
}

Response:
{
  "success": true,
  "processed": 1,
  "failed": 0,
  "errors": [],
  "erp_references": [
    {
      "pointing_id": 123456,
      "erp_transaction_id": "TRX-789012",
      "erp_document_number": "DOC-456789"
    }
  ]
}
```

#### **8.2.3 Consultar Estoque de Materiais**
```http
GET /api/v1/material-stock/{product_code}
Headers:
  Authorization: Bearer {api_token}

Response:
{
  "success": true,
  "data": {
    "product_code": "1.23456.789.123456",
    "description": "Tecido Algodão 200g/m² Branco",
    "current_stock": 12500.50,
    "unit": "metros",
    "allocated_stock": 7500.00,
    "available_stock": 5000.50,
    "minimum_stock": 10000.00,
    "reorder_point": 15000.00,
    "lead_time_days": 7,
    "last_receipt": {
      "date": "2024-01-10",
      "quantity": 5000.00,
      "supplier": "Tecelagem Norte Ltda"
    },
    "next_receipt": {
      "date": "2024-01-20",
      "expected_quantity": 10000.00,
      "supplier": "Tecelagem Norte Ltda"
    }
  }
}
```

### **8.3 Serviço de Sincronização (Python)**

#### **8.3.1 Classe Principal de Sincronização**
```python
# services/erp_sync_service.py
import requests
import logging
from datetime import datetime, timedelta
from typing import List, Dict, Optional
from models import db, ProductionOrder, SyncLog, ProductionPointing

logger = logging.getLogger(__name__)

class ERPSyncService:
    def __init__(self, base_url: str, api_key: str, api_token: str):
        self.base_url = base_url.rstrip('/')
        self.api_key = api_key
        self.api_token = api_token
        self.headers = {
            'Authorization': f'Bearer {api_token}',
            'X-API-Key': api_key,
            'Content-Type': 'application/json'
        }
        self.timeout = 30  # segundos
    
    def sync_production_orders(self, from_date: Optional[datetime] = None) -> Dict:
        """
        Sincronizar Ordens de Produção do ERP para o sistema local
        """
        sync_log = SyncLog(
            sync_type='production_orders',
            direction='import',
            started_at=datetime.utcnow()
        )
        
        try:
            # Parâmetros da requisição
            params = {}
            if from_date:
                params['from_date'] = from_date.strftime('%Y-%m-%d')
            
            # Buscar OPs do ERP
            response = requests.get(
                f'{self.base_url}/api/v1/production-orders',
                headers=self.headers,
                params=params,
                timeout=self.timeout
            )
            
            if response.status_code != 200:
                raise Exception(f'ERP API error: {response.status_code}')
            
            data = response.json()
            
            if not data.get('success'):
                raise Exception(f'ERP API returned error: {data.get("error")}')
            
            # Processar cada OP
            processed = 0
            successful = 0
            failed = 0
            errors = []
            
            for op_data in data.get('data', []):
                processed += 1
                
                try:
                    self._process_production_order(op_data)
                    successful += 1
                    
                except Exception as e:
                    failed += 1
                    error_msg = f"OP {op_data.get('op_number')}: {str(e)}"
                    errors.append(error_msg)
                    logger.error(error_msg)
            
            # Atualizar log de sincronização
            sync_log.status = 'partial' if failed > 0 else 'success'
            sync_log.records_processed = processed
            sync_log.records_successful = successful
            sync_log.records_failed = failed
            sync_log.error_details = {'errors': errors} if errors else None
            
            return {
                'success': True,
                'processed': processed,
                'successful': successful,
                'failed': failed,
                'errors': errors
            }
            
        except Exception as e:
            logger.error(f"Erro na sincronização de OPs: {str(e)}")
            sync_log.status = 'error'
            sync_log.error_message = str(e)
            raise
            
        finally:
            sync_log.finished_at = datetime.utcnow()
            if sync_log.started_at and sync_log.finished_at:
                sync_log.duration_seconds = (
                    sync_log.finished_at - sync_log.started_at
                ).total_seconds()
            
            db.session.add(sync_log)
            db.session.commit()
    
    def _process_production_order(self, op_data: Dict):
        """
        Processar dados de uma OP do ERP
        """
        op_number = op_data['op_number']
        
        # Verificar se OP já existe
        existing_op = ProductionOrder.query.filter_by(op_number=op_number).first()
        
        if existing_op:
            # Atualizar OP existente (se não estiver em produção)
            if existing_op.status in ['pending', 'released']:
                existing_op.product_code = op_data['product_code']
                existing_op.product_description = op_data['product_description']
                existing_op.programmed_quantity = op_data['programmed_quantity']
                existing_op.loaded_quantity = op_data['loaded_quantity']
                existing_op.start_date = datetime.strptime(op_data['start_date'], '%Y-%m-%d').date()
                existing_op.end_date = datetime.strptime(op_data['end_date'], '%Y-%m-%d').date()
                existing_op.priority = op_data.get('priority', 1)
                existing_op.status = op_data['status']
                existing_op.erp_sync_status = 'synced'
                existing_op.erp_sync_date = datetime.utcnow()
                
                logger.info(f"OP {op_number} atualizada do ERP")
        else:
            # Criar nova OP
            new_op = ProductionOrder(
                op_number=op_number,
                erp_id=op_data.get('erp_id'),
                product_code=op_data['product_code'],
                product_description=op_data['product_description'],
                programmed_quantity=op_data['programmed_quantity'],
                loaded_quantity=op_data['loaded_quantity'],
                start_date=datetime.strptime(op_data['start_date'], '%Y-%m-%d').date(),
                end_date=datetime.strptime(op_data['end_date'], '%Y-%m-%d').date(),
                priority=op_data.get('priority', 1),
                status=op_data['status'],
                customer_code=op_data.get('customer_code'),
                customer_name=op_data.get('customer_name'),
                erp_sync_status='synced',
                erp_sync_date=datetime.utcnow(),
                created_at=datetime.utcnow()
            )
            
            db.session.add(new_op)
            logger.info(f"OP {op_number} criada a partir do ERP")
        
        db.session.commit()
    
    def send_pointing_to_erp(self, pointing_ids: List[int]) -> Dict:
        """
        Enviar apontamentos para o ERP
        """
        sync_log = SyncLog(
            sync_type='production_pointing',
            direction='export',
            started_at=datetime.utcnow()
        )
        
        try:
            # Buscar apontamentos do banco local
            pointings = ProductionPointing.query.filter(
                ProductionPointing.id.in_(pointing_ids),
                ProductionPointing.sync_status == 'pending'
            ).all()
            
            if not pointings:
                return {'success': True, 'message': 'Nenhum apontamento pendente'}
            
            # Preparar dados para envio
            pointing_data = []
            for pointing in pointings:
                pointing_dict = self._prepare_pointing_data(pointing)
                pointing_data.append(pointing_dict)
            
            # Enviar para o ERP
            payload = {'pointing_data': pointing_data}
            
            response = requests.post(
                f'{self.base_url}/api/v1/production-pointing',
                headers=self.headers,
                json=payload,
                timeout=self.timeout
            )
            
            if response.status_code != 200:
                raise Exception(f'ERP API error: {response.status_code}')
            
            response_data = response.json()
            
            if not response_data.get('success'):
                raise Exception(f'ERP API returned error: {response_data.get("error")}')
            
            # Atualizar status dos apontamentos
            erp_references = response_data.get('erp_references', [])
            
            for ref in erp_references:
                pointing = ProductionPointing.query.get(ref['pointing_id'])
                if pointing:
                    pointing.sync_status = 'synced'
                    pointing.erp_transaction_id = ref.get('erp_transaction_id')
                    pointing.erp_document_number = ref.get('erp_document_number')
                    pointing.sync_date = datetime.utcnow()
            
            db.session.commit()
            
            # Atualizar log de sincronização
            sync_log.status = 'success'
            sync_log.records_processed = len(pointings)
            sync_log.records_successful = len(pointings)
            sync_log.records_failed = 0
            
            return {
                'success': True,
                'processed': len(pointings),
                'erp_references': erp_references
            }
            
        except Exception as e:
            logger.error(f"Erro ao enviar apontamentos para ERP: {str(e)}")
            sync_log.status = 'error'
            sync_log.error_message = str(e)
            
            # Marcar apontamentos como erro
            for pointing in pointings:
                pointing.sync_status = 'error'
                pointing.sync_notes = str(e)
            
            db.session.commit()
            raise
            
        finally:
            sync_log.finished_at = datetime.utcnow()
            if sync_log.started_at and sync_log.finished_at:
                sync_log.duration_seconds = (
                    sync_log.finished_at - sync_log.started_at
                ).total_seconds()
            
            db.session.add(sync_log)
            db.session.commit()
    
    def _prepare_pointing_data(self, pointing: ProductionPointing) -> Dict:
        """
        Preparar dados do apontamento para envio ao ERP
        """
        # Buscar informações relacionadas
        op = pointing.production_order
        machine = pointing.machine
        user = pointing.user
        
        # Buscar paradas associadas
        stops = pointing.machine_stops.filter_by(stop_status='resolved').all()
        
        stop_events = []
        for stop in stops:
            stop_reason = stop.stop_reason
            stop_events.append({
                'stop_reason': stop_reason.code,
                'start_time': stop.start_time.isoformat(),
                'end_time': stop.end_time.isoformat() if stop.end_time else None,
                'duration_minutes': stop.duration_minutes,
                'notes': stop.notes
            })
        
        return {
            'op_number': op.op_number,
            'machine_code': machine.code,
            'operator_code': user.employee_code,
            'start_time': pointing.start_time.isoformat(),
            'end_time': pointing.end_time.isoformat(),
            'produced_quantity': float(pointing.produced_quantity),
            'good_quantity': float(pointing.good_quantity) if pointing.good_quantity else None,
            'rejected_quantity': float(pointing.rejected_quantity) if pointing.rejected_quantity else 0,
            'production_stage': pointing.production_stage,
            'standard_speed': float(pointing.standard_speed) if pointing.standard_speed else None,
            'actual_speed': float(pointing.actual_speed) if pointing.actual_speed else None,
            'efficiency': float(pointing.efficiency) if pointing.efficiency else None,
            'notes': pointing.notes,
            'stop_events': stop_events
        }
    
    def check_material_availability(self, product_code: str) -> Dict:
        """
        Verificar disponibilidade de material no ERP
        """
        try:
            response = requests.get(
                f'{self.base_url}/api/v1/material-stock/{product_code}',
                headers=self.headers,
                timeout=self.timeout
            )
            
            if response.status_code != 200:
                return {'available': False, 'error': f'API error: {response.status_code}'}
            
            data = response.json()
            
            if not data.get('success'):
                return {'available': False, 'error': data.get('error')}
            
            stock_data = data.get('data', {})
            available_stock = stock_data.get('available_stock', 0)
            minimum_stock = stock_data.get('minimum_stock', 0)
            
            return {
                'available': available_stock > minimum_stock,
                'available_stock': available_stock,
                'minimum_stock': minimum_stock,
                'details': stock_data
            }
            
        except Exception as e:
            logger.error(f"Erro ao verificar estoque: {str(e)}")
            return {'available': False, 'error': str(e)}
```

#### **8.3.2 Tarefa Celery para Sincronização Automática**
```python
# tasks/sync_tasks.py
from celery import Celery
from datetime import datetime, timedelta
from services.erp_sync_service import ERPSyncService
from config import Config

celery = Celery(__name__, broker=Config.CELERY_BROKER_URL)

@celery.task(name='sync_production_orders')
def sync_production_orders_task():
    """
    Tarefa para sincronizar OPs do ERP periodicamente
    """
    try:
        sync_service = ERPSyncService(
            base_url=Config.ERP_API_URL,
            api_key=Config.ERP_API_KEY,
            api_token=Config.ERP_API_TOKEN
        )
        
        # Sincronizar OPs dos últimos 7 dias
        from_date = datetime.utcnow() - timedelta(days=7)
        
        result = sync_service.sync_production_orders(from_date=from_date)
        
        logger.info(f"Sincronização de OPs concluída: {result}")
        
        return result
        
    except Exception as e:
        logger.error(f"Erro na tarefa de sincronização de OPs: {str(e)}")
        raise

@celery.task(name='send_pending_pointings')
def send_pending_pointings_task():
    """
    Tarefa para enviar apontamentos pendentes para o ERP
    """
    try:
        from models import ProductionPointing
        
        # Buscar apontamentos pendentes
        pending_pointings = ProductionPointing.query.filter_by(
            sync_status='pending'
        ).limit(100).all()  # Limitar para não sobrecarregar
        
        if not pending_pointings:
            return {'success': True, 'message': 'Nenhum apontamento pendente'}
        
        pointing_ids = [p.id for p in pending_pointings]
        
        sync_service = ERPSyncService(
            base_url=Config.ERP_API_URL,
            api_key=Config.ERP_API_KEY,
            api_token=Config.ERP_API_TOKEN
        )
        
        result = sync_service.send_pointing_to_erp(pointing_ids)
        
        logger.info(f"Apontamentos enviados para ERP: {result}")
        
        return result
        
    except Exception as e:
        logger.error(f"Erro ao enviar apontamentos para ERP: {str(e)}")
        raise

@celery.task(name='sync_daily_summary')
def sync_daily_summary_task():
    """
    Tarefa para enviar resumo diário para o ERP
    """
    try:
        from models import db, ProductionKPI, ProductionPointing
        from datetime import date
        
        today = date.today()
        
        # Calcular totais do dia
        total_produced = db.session.query(
            db.func.sum(ProductionPointing.produced_quantity)
        ).filter(
            db.func.date(ProductionPointing.start_time) == today
        ).scalar() or 0
        
        total_stops = db.session.query(
            db.func.sum(db.func.extract('epoch', MachineStop.duration_minutes) / 60)
        ).filter(
            db.func.date(MachineStop.start_time) == today,
            MachineStop.stop_status == 'resolved'
        ).scalar() or 0
        
        # Preparar resumo
        summary = {
            'date': today.isoformat(),
            'total_produced': float(total_produced),
            'total_stop_hours': float(total_stops),
            'average_efficiency': 0,  # Seria calculado
            'notes': 'Resumo automático do Production Pointer'
        }
        
        # Aqui iria a chamada para API do ERP
        # Por enquanto, apenas log
        logger.info(f"Resumo diário gerado: {summary}")
        
        return summary
        
    except Exception as e:
        logger.error(f"Erro ao gerar resumo diário: {str(e)}")
        raise
```

#### **8.3.3 Configuração do Celery Beat (Agendador)**
```python
# celery_config.py
from datetime import timedelta

CELERY_BEAT_SCHEDULE = {
    # Sincronizar OPs a cada 5 minutos
    'sync-production-orders-every-5-minutes': {
        'task': 'sync_production_orders',
        'schedule': timedelta(minutes=5),
        'options': {'queue': 'erp_sync'}
    },
    
    # Enviar apontamentos a cada 2 minutos
    'send-pending-pointings-every-2-minutes': {
        'task': 'send_pending_pointings',
        'schedule': timedelta(minutes=2),
        'options': {'queue': 'erp_sync'}
    },
    
    # Enviar resumo diário às 23:30
    'send-daily-summary-at-2330': {
        'task': 'sync_daily_summary',
        'schedule': {'hour': 23, 'minute': 30, 'day_of_week': '*'},
        'options': {'queue': 'reports'}
    },
    
    # Calcular KPIs a cada hora
    'calculate-hourly-kpis': {
        'task': 'calculate_kpis',
        'schedule': timedelta(hours=1),
        'options': {'queue': 'calculations'}
    },
    
    # Limpar dados antigos diariamente às 2:00
    'clean-old-data-at-0200': {
        'task': 'clean_old_data',
        'schedule': {'hour': 2, 'minute': 0, 'day_of_week': '*'},
        'options': {'queue': 'maintenance'}
    }
}

CELERY_TASK_ROUTES = {
    'sync_production_orders': {'queue': 'erp_sync'},
    'send_pending_pointings': {'queue': 'erp_sync'},
    'sync_daily_summary': {'queue': 'reports'},
    'calculate_kpis': {'queue': 'calculations'},
    'clean_old_data': {'queue': 'maintenance'},
    'generate_report': {'queue': 'reports'},
    'send_notification': {'queue': 'notifications'}
}
```

### **8.4 Configuração de Filas Redis para Celery**

```bash
#!/bin/bash
# configure_redis_queues.sh

# Configurar filas Redis separadas para diferentes tipos de tarefas
# No arquivo de configuração do Celery (celery_config.py):

"""
# Broker URL com database separados
BROKER_URL = 'redis://:R3d!sS3nh@2024@localhost:6379/0'
RESULT_BACKEND = 'redis://:R3d!sS3nh@2024@localhost:6379/1'

# Configurar filas separadas
task_routes = {
    'tasks.erp_sync.*': {'queue': 'erp_sync'},
    'tasks.reports.*': {'queue': 'reports'},
    'tasks.calculations.*': {'queue': 'calculations'},
    'tasks.notifications.*': {'queue': 'notifications'},
    'tasks.maintenance.*': {'queue': 'maintenance'},
}
"""

# Criar script de inicialização do Celery
sudo tee /etc/systemd/system/celery.service << 'EOF'
[Unit]
Description=Celery Service for Production Pointer
After=network.target redis-server.service postgresql.service

[Service]
Type=forking
User=production
Group=production
WorkingDirectory=/opt/productionpointer/app
Environment="PATH=/opt/productionpointer/venv/bin"
ExecStart=/opt/productionpointer/venv/bin/celery -A app.celery multi start 4 \
    -l info \
    --pidfile=/opt/productionpointer/data/celery/%n.pid \
    --logfile=/opt/productionpointer/data/logs/celery/%n%I.log \
    --concurrency=2 \
    -Q:1-2 erp_sync,calculations \
    -Q:3 reports,notifications \
    -Q:4 maintenance
ExecStop=/opt/productionpointer/venv/bin/celery -A app.celery multi stop 4 \
    --pidfile=/opt/productionpointer/data/celery/%n.pid
ExecReload=/opt/productionpointer/venv/bin/celery -A app.celery multi restart 4 \
    --pidfile=/opt/productionpointer/data/celery/%n.pid \
    --logfile=/opt/productionpointer/data/logs/celery/%n%I.log
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo tee /etc/systemd/system/celerybeat.service << 'EOF'
[Unit]
Description=Celery Beat Service for Production Pointer
After=network.target redis-server.service postgresql.service celery.service

[Service]
Type=simple
User=production
Group=production
WorkingDirectory=/opt/productionpointer/app
Environment="PATH=/opt/productionpointer/venv/bin"
ExecStart=/opt/productionpointer/venv/bin/celery -A app.celery beat \
    -l info \
    --pidfile=/opt/productionpointer/data/celery/beat.pid \
    --logfile=/opt/productionpointer/data/logs/celery/beat.log \
    --schedule=/opt/productionpointer/data/celery/celerybeat-schedule
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# Criar diretórios
sudo mkdir -p /opt/productionpointer/data/{celery,logs/celery}
sudo chown -R production:production /opt/productionpointer/data

# Habilitar e iniciar serviços
sudo systemctl daemon-reload
sudo systemctl enable celery.service celerybeat.service
sudo systemctl start celery.service celerybeat.service

# Verificar status
sudo systemctl status celery.service celerybeat.service
```

### **8.5 Estratégia de Sincronização em Caso de Falha**

#### **8.5.1 Retry com Backoff Exponencial**
```python
def sync_with_retry(task_func, max_retries=5, initial_delay=1):
    """
    Executar tarefa com retry automático
    """
    delay = initial_delay
    
    for attempt in range(max_retries):
        try:
            return task_func()
            
        except requests.exceptions.ConnectionError as e:
            if attempt == max_retries - 1:
                raise
            
            logger.warning(f"Falha de conexão (tentativa {attempt + 1}/{max_retries}): {str(e)}")
            time.sleep(delay)
            delay *= 2  # Backoff exponencial
            
        except Exception as e:
            logger.error(f"Erro na sincronização: {str(e)}")
            raise
```

#### **8.5.2 Modo de Operação Degradado**
```
Quando: ERP indisponível por período prolongado

Estratégia:
1. Continuar operação normal no sistema local
2. Armazenar dados pendentes em fila persistente
3. Mostrar indicador "ERP Offline" no dashboard
4. Aumentar capacidade de armazenamento local
5. Quando ERP retornar:
   - Sincronizar dados em lote
   - Validar consistência
   - Resolver conflitos manualmente se necessário
6. Manter histórico de período offline para auditoria
```

#### **8.5.3 Validação de Consistência Pós-Sincronização**
```python
def validate_sync_consistency():
    """
    Validar consistência entre sistema local e ERP
    """
    inconsistencies = []
    
    # Verificar OPs com status diferente
    local_ops = ProductionOrder.query.all()
    
    for op in local_ops:
        # Buscar status no ERP (simulação)
        erp_status = get_erp_op_status(op.op_number)
        
        if erp_status and erp_status != op.status:
            inconsistencies.append({
                'type': 'status_mismatch',
                'op_number': op.op_number,
                'local_status': op.status,
                'erp_status': erp_status,
                'suggested_action': 'reconcile_with_erp'
            })
    
    # Verificar quantidades divergentes
    pointings = ProductionPointing.query.filter_by(sync_status='synced').all()
    
    for pointing in pointings:
        if pointing.erp_transaction_id:
            erp_quantity = get_erp_pointing_quantity(pointing.erp_transaction_id)
            
            if erp_quantity and abs(erp_quantity - pointing.produced_quantity) > 0.01:
                inconsistencies.append({
                    'type': 'quantity_mismatch',
                    'pointing_id': pointing.id,
                    'local_quantity': pointing.produced_quantity,
                    'erp_quantity': erp_quantity,
                    'difference': erp_quantity - pointing.produced_quantity
                })
    
    return inconsistencies
```

---

## **9. SEGURANÇA E HARDENING**

### **9.1 Estratégia de Segurança em Camadas**

```
┌─────────────────────────────────────────────────────────────┐
│                    CAMADAS DE SEGURANÇA                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  CAMADA 7: DADOS                                            │
│  • Criptografia em repouso (LUKS, database)                │
│  • Criptografia em trânsito (TLS 1.3)                      │
│  • Máscara de dados sensíveis                              │
│  • Backup criptografado                                     │
│                                                             │
│  CAMADA 6: APLICAÇÃO                                        │
│  • Autenticação JWT com refresh tokens                     │
│  • Rate limiting por IP/usuário                            │
│  • Validação de entrada (SQL injection, XSS)               │
│  • CSRF protection                                         │
│  • Headers de segurança (CSP, HSTS)                        │
│                                                             │
│  CAMADA 5: SESSÃO                                           │
│  • Timeout de sessão configurável                          │
│  • Single session por usuário (opcional)                   │
│  • Logout em todos os dispositivos                         │
│  • Detecção de atividade suspeita                          │
│                                                             │
│  CAMADA 4: REDE                                             │
│  • Firewall (UFW) com regras mínimas                       │
│  • VLAN separada para dispositivos IoT                     │
│  • VPN para acesso remoto                                  │
│  • IDS/IPS (Suricata/Snort)                                │
│                                                             │
│  CAMADA 3: SISTEMA                                          │
│  • Hardening do SO (CIS benchmarks)                        │
│  • Atualizações automáticas de segurança                   │
│  • Logs centralizados                                      │
│  • Monitoramento de integridade de arquivos                │
│                                                             │
│  CAMADA 2: FÍSICA                                           │
│  • Servidor em sala trancada                               │
│  • UPS com autonomia                                       │
│  • Controle de acesso físico                               │
│  • Câmeras de segurança                                    │
│                                                             │
│  CAMADA 1: HUMANA                                           │
│  • Treinamento de segurança                                │
│  • Políticas de uso aceitável                              │
│  • Senhas fortes e 2FA                                     │
│  • Procedimentos de incidente                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### **9.2 Configuração de Segurança do Sistema Operacional**

#### **9.2.1 Hardening do Ubuntu Server**
```bash
#!/bin/bash
# security_hardening.sh

# 1. ATUALIZAÇÕES DE SEGURANÇA
sudo apt update
sudo apt install unattended-upgrades apt-listchanges
sudo dpkg-reconfigure --priority=low unattended-upgrades

# Configurar atualizações automáticas
sudo tee /etc/apt/apt.conf.d/50unattended-upgrades << 'EOF'
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "02:00";
EOF

# 2. CONFIGURAR SYSCTL (PARÂMETROS DO KERNEL)
sudo tee /etc/sysctl.d/99-security.conf << 'EOF'
# Proteção contra spoofing
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Ignorar ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0

# Não enviar redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Proteção contra SYN flood
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2

# Log de pacotes suspeitos
net.ipv4.conf.all.log_martians = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Proteção contra ataques de roteamento
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Proteção IP spoofing
net.ipv4.conf.all.secure_redirects = 1
net.ipv4.conf.default.secure_redirects = 1

# Ignorar broadcasts
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Ignorar ping
net.ipv4.icmp_echo_ignore_all = 1

# Proteção contra smurf attacks
net.ipv4.icmp_ignore_bogus_error_messages = 1

# Aumentar limites de conexão
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 5000
EOF

sudo sysctl -p /etc/sysctl.d/99-security.conf

# 3. CONFIGURAR SSH HARDENING
sudo tee /etc/ssh/sshd_config.d/99-hardening.conf << 'EOF'
# Porta não padrão (opcional)
# Port 2222

# Protocolo 2 apenas
Protocol 2

# Desabilitar root login
PermitRootLogin no

# Desabilitar senha vazia
PermitEmptyPasswords no

# Autenticação por senha (desabilitar se usar chaves)
PasswordAuthentication yes

# Configurações de segurança
X11Forwarding no
AllowTcpForwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
MaxSessions 2
LoginGraceTime 60

# Ciphers seguros
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

# AllowUsers production admin-prod
EOF

sudo systemctl restart sshd

# 4. INSTALAR E CONFIGURAR FAIL2BAN
sudo apt install -y fail2ban

# Configuração específica para aplicação
sudo tee /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
ignoreip = 127.0.0.1/8 10.0.0.0/24 192.168.1.0/24
bantime = 3600
findtime = 600
maxretry = 5
backend = auto
usedns = warn
logencoding = auto
enabled = true
mode = normal
filter = %(__name__)s[mode=%(mode)s]
destemail = admin@empresa.com.br
sender = fail2ban@production-server
action = %(action_)s
banaction = ufw
protocol = tcp
chain = INPUT
port = 0:65535
fail2ban_agent = Fail2Ban/%(fail2ban_version)s

[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
maxretry = 3

[nginx-http-auth]
enabled = true
port = http,https
logpath = /opt/productionpointer/data/logs/nginx_access.log
maxretry = 3

[nginx-botsearch]
enabled = true
port = http,https
logpath = /opt/productionpointer/data/logs/nginx_access.log
maxretry = 5

[nginx-proxy]
enabled = true
port = http,https
logpath = /opt/productionpointer/data/logs/nginx_access.log
maxretry = 3

[production-pointer-auth]
enabled = true
port = http,https
filter = production-pointer-auth
logpath = /opt/productionpointer/data/logs/app_stderr.log
maxretry = 5
bantime = 86400
findtime = 3600
EOF

# Criar filtro customizado
sudo tee /etc/fail2ban/filter.d/production-pointer-auth.conf << 'EOF'
[Definition]
failregex = ^.*Authentication failed for user.*IP <HOST>.*$
ignoreregex = 
EOF

sudo systemctl restart fail2ban
sudo systemctl enable fail2ban

# 5. CONFIGURAR LOGROTATE PARA APLICAÇÃO
sudo tee /etc/logrotate.d/production-pointer << 'EOF'
/opt/productionpointer/data/logs/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 640 production production
    sharedscripts
    postrotate
        sudo supervisorctl restart productionpointer > /dev/null 2>&1 || true
        sudo supervisorctl restart celery_worker > /dev/null 2>&1 || true
        sudo supervisorctl restart celery_beat > /dev/null 2>&1 || true
    endscript
}
EOF

# 6. CONFIGURAR AUDITD (AUDITORIA DO SISTEMA)
sudo apt install -y auditd audispd-plugins

sudo tee /etc/audit/rules.d/99-production-pointer.rules << 'EOF'
# Monitorar acesso a arquivos da aplicação
-w /opt/productionpointer/app -p wa -k production_pointer_app
-w /opt/productionpointer/data -p wa -k production_pointer_data
-w /etc/nginx/sites-available/productionpointer -p wa -k nginx_config
-w /etc/postgresql/15/main/postgresql.conf -p wa -k postgresql_config
-w /etc/redis/redis.conf -p wa -k redis_config

# Monitorar tentativas de login
-a always,exit -F arch=b64 -S execve -C uid!=euid -F euid=0 -k privilege_escalation
-a always,exit -F arch=b64 -S execve -C gid!=egid -F egid=0 -k privilege_escalation

# Monitorar alterações em senhas e grupos
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k identity
EOF

sudo auditctl -R /etc/audit/rules.d/99-production-pointer.rules
sudo systemctl restart auditd
sudo systemctl enable auditd

# 7. INSTALAR E CONFIGURAR RKHUNTER (ROOTKIT DETECTION)
sudo apt install -y rkhunter

sudo rkhunter --propupd  # Atualizar banco de dados
sudo rkhunter --check --skip-keypress  # Verificação inicial

# Agendar verificação diária
sudo tee /etc/cron.daily/rkhunter << 'EOF'
#!/bin/bash
/usr/bin/rkhunter --cronjob --report-warnings-only
EOF

sudo chmod +x /etc/cron.daily/rkhunter

# 8. CONFIGURAR TCPWRAPPERS (HOST-BASED ACCESS CONTROL)
sudo tee /etc/hosts.allow << 'EOF'
# Permitir apenas redes internas
sshd: 10.0.0.0/24, 192.168.1.0/24
ALL: 127.0.0.1
EOF

sudo tee /etc/hosts.deny << 'EOF'
# Negar todo o resto
ALL: ALL
EOF

# 9. CONFIGURAR PERMISSÕES DE ARQUIVOS
sudo chmod 750 /opt/productionpointer/app
sudo chmod 640 /opt/productionpointer/app/.env
sudo chmod 750 /opt/productionpointer/venv
sudo chown -R production:production /opt/productionpointer

# 10. INSTALAR E CONFIGURAR CLAMAV (ANTIVÍRUS)
sudo apt install -y clamav clamav-daemon

# Atualizar banco de vírus
sudo freshclam

# Configurar varredura diária
sudo tee /etc/cron.daily/clamav-scan << 'EOF'
#!/bin/bash
/usr/bin/clamscan -r -i --exclude-dir="^/sys" --exclude-dir="^/proc" --exclude-dir="^/dev" /opt/productionpointer/app
EOF

sudo chmod +x /etc/cron.daily/clamav-scan

echo "Hardening básico concluído!"
echo "Recomendações adicionais:"
echo "1. Configurar 2FA para SSH"
echo "2. Implementar VPN para acesso remoto"
echo "3. Configurar monitoramento (Zabbix/Nagios)"
echo "4. Fazer pentest regular"
```

### **9.3 Segurança na Aplicação Flask**

#### **9.3.1 Configuração de Segurança no Flask**
```python
# app/security.py
from flask import Flask
from flask_talisman import Talisman
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
from flask_cors import CORS
import os

def configure_security(app: Flask):
    """
    Configurar todas as medidas de segurança da aplicação
    """
    
    # 1. TLS/HTTPS Enforcement (Flask-Talisman)
    csp = {
        'default-src': "'self'",
        'script-src': ["'self'", "'unsafe-inline'", "cdnjs.cloudflare.com"],
        'style-src': ["'self'", "'unsafe-inline'", "fonts.googleapis.com"],
        'font-src': ["'self'", "fonts.gstatic.com"],
        'img-src': ["'self'", "data:", "blob:"],
        'connect-src': ["'self'", "ws:", "wss:"],
        'frame-ancestors': "'none'",
        'base-uri': "'self'",
        'form-action': "'self'"
    }
    
    Talisman(
        app,
        force_https=True,
        force_https_permanent=True,
        strict_transport_security=True,
        strict_transport_security_max_age=31536000,
        strict_transport_security_include_subdomains=True,
        strict_transport_security_preload=True,
        frame_options='DENY',
        content_security_policy=csp,
        content_security_policy_report_uri='/csp-violation-report',
        referrer_policy='strict-origin-when-cross-origin',
        session_cookie_secure=True,
        session_cookie_http_only=True,
        session_cookie_samesite='Lax'
    )
    
    # 2. Rate Limiting (Flask-Limiter)
    limiter = Limiter(
        app=app,
        key_func=get_remote_address,
        default_limits=["200 per day", "50 per hour"],
        storage_uri="redis://localhost:6379/2",
        strategy="fixed-window",
        headers_enabled=True
    )
    
    # Limites específicos
    limiter.limit("10 per minute")(app.route('/api/login', methods=['POST']))
    limiter.limit("60 per minute")(app.route('/api/production/*'))
    limiter.limit("5 per minute")(app.route('/api/admin/*'))
    
    # 3. CORS Configuration
    CORS(
        app,
        resources={
            r"/api/*": {
                "origins": [
                    "https://producao.empresa.com.br",
                    "http://10.0.0.*",
                    "http://192.168.1.*"
                ],
                "methods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
                "allow_headers": ["Content-Type", "Authorization"],
                "expose_headers": ["X-RateLimit-Limit", "X-RateLimit-Remaining"],
                "supports_credentials": True,
                "max_age": 600
            }
        }
    )
    
    # 4. Security Headers Adicionais
    @app.after_request
    def add_security_headers(response):
        response.headers['X-Content-Type-Options'] = 'nosniff'
        response.headers['X-Frame-Options'] = 'DENY'
        response.headers['X-XSS-Protection'] = '1; mode=block'
        response.headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
        response.headers['Permissions-Policy'] = 'geolocation=(), microphone=(), camera=()'
        
        # Headers customizados para a aplicação
        response.headers['X-Application-Name'] = 'Production Pointer Pro'
        response.headers['X-Application-Version'] = '1.0.0'
        
        return response
    
    # 5. Proteção contra ataques comuns
    app.config.update(
        # Proteção contra CSRF
        WTF_CSRF_ENABLED=True,
        WTF_CSRF_SECRET_KEY=os.getenv('CSRF_SECRET_KEY'),
        
        # Proteção contra clickjacking
        SESSION_COOKIE_SAMESITE='Lax',
        SESSION_COOKIE_SECURE=True,
        SESSION_COOKIE_HTTPONLY=True,
        
        # Prevenir MIME sniffing
        JSONIFY_PRETTYPRINT_REGULAR=False,
        
        # Limitar tamanho de upload
        MAX_CONTENT_LENGTH=16 * 1024 * 1024,  # 16MB
        
        # Configurações de sessão
        PERMANENT_SESSION_LIFETIME=timedelta(hours=8),
        SESSION_REFRESH_EACH_REQUEST=True,
        
        # Logging de segurança
        SECURITY_LOGGING_ENABLED=True
    )
    
    return app, limiter

# app/auth/security.py
import hashlib
import hmac
import base64
import json
from datetime import datetime, timedelta
import secrets
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError
import jwt

class SecurityManager:
    def __init__(self):
        self.ph = PasswordHasher(
            time_cost=3,          # Número de iterações
            memory_cost=65536,    # 64MB de memória
            parallelism=4,        # 4 threads
            hash_len=32,
            salt_len=16
        )
        
        # Chaves para JWT (deveriam estar em variáveis de ambiente)
        self.jwt_secret_key = os.getenv('JWT_SECRET_KEY', secrets.token_urlsafe(64))
        self.jwt_refresh_secret_key = os.getenv('JWT_REFRESH_SECRET_KEY', secrets.token_urlsafe(64))
        
    def hash_password(self, password: str) -> str:
        """Hash de senha usando Argon2"""
        return self.ph.hash(password)
    
    def verify_password(self, password: str, password_hash: str) -> bool:
        """Verificar senha"""
        try:
            return self.ph.verify(password_hash, password)
        except VerifyMismatchError:
            return False
    
    def create_access_token(self, user_id: int, user_type: str, permissions: list) -> str:
        """Criar JWT access token"""
        payload = {
            'user_id': user_id,
            'user_type': user_type,
            'permissions': permissions,
            'exp': datetime.utcnow() + timedelta(minutes=30),  # 30 minutos
            'iat': datetime.utcnow(),
            'type': 'access',
            'jti': secrets.token_urlsafe(16)  # ID único do token
        }
        
        return jwt.encode(payload, self.jwt_secret_key, algorithm='HS256')
    
    def create_refresh_token(self, user_id: int) -> str:
        """Criar JWT refresh token"""
        payload = {
            'user_id': user_id,
            'exp': datetime.utcnow() + timedelta(days=7),  # 7 dias
            'iat': datetime.utcnow(),
            'type': 'refresh',
            'jti': secrets.token_urlsafe(16)
        }
        
        return jwt.encode(payload, self.jwt_refresh_secret_key, algorithm='HS256')
    
    def verify_token(self, token: str, token_type: str = 'access') -> dict:
        """Verificar e decodificar JWT"""
        try:
            secret_key = self.jwt_secret_key if token_type == 'access' else self.jwt_refresh_secret_key
            
            payload = jwt.decode(
                token,
                secret_key,
                algorithms=['HS256'],
                options={'require': ['exp', 'iat', 'type', 'jti']}
            )
            
            if payload.get('type') != token_type:
                raise jwt.InvalidTokenError('Token type mismatch')
            
            return payload
            
        except jwt.ExpiredSignatureError:
            raise ValueError('Token expirado')
        except jwt.InvalidTokenError as e:
            raise ValueError(f'Token inválido: {str(e)}')
    
    def generate_csrf_token(self) -> str:
        """Gerar token CSRF"""
        return secrets.token_urlsafe(32)
    
    def validate_csrf_token(self, token: str, stored_token: str) -> bool:
        """Validar token CSRF"""
        return hmac.compare_digest(token, stored_token)
    
    def sanitize_input(self, input_data: str) -> str:
        """Sanitizar entrada de dados"""
        import html
        import re
        
        # Remover tags HTML/JavaScript
        sanitized = html.escape(input_data)
        
        # Remover caracteres de controle
        sanitized = re.sub(r'[\x00-\x1F\x7F]', '', sanitized)
        
        # Limitar comprimento
        max_length = 1000
        if len(sanitized) > max_length:
            sanitized = sanitized[:max_length]
        
        return sanitized
    
    def log_security_event(self, event_type: str, user_id: int = None, 
                          ip_address: str = None, details: dict = None):
        """Registrar evento de segurança"""
        from models import SecurityLog
        
        log = SecurityLog(
            event_type=event_type,
            user_id=user_id,
            ip_address=ip_address,
            user_agent=request.headers.get('User-Agent'),
            details=details or {},
            created_at=datetime.utcnow()
        )
        
        db.session.add(log)
        db.session.commit()
```

#### **9.3.2 Middleware de Segurança**
```python
# app/middleware/security_middleware.py
from functools import wraps
from flask import request, jsonify, g
from flask_limiter import RateLimitExceeded
import time

class SecurityMiddleware:
    
    @staticmethod
    def rate_limit_exceeded_handler(e):
        """Handler para limite de taxa excedido"""
        return jsonify({
            'error': 'rate_limit_exceeded',
            'message': 'Muitas requisições. Tente novamente mais tarde.',
            'retry_after': f'{e.retry_after} segundos'
        }), 429
    
    @staticmethod
    def require_authentication(f):
        """Decorator para requisições autenticadas"""
        @wraps(f)
        def decorated_function(*args, **kwargs):
            auth_header = request.headers.get('Authorization')
            
            if not auth_header:
                return jsonify({'error': 'missing_authorization'}), 401
            
            try:
                # Formato: Bearer <token>
                token = auth_header.split(' ')[1]
                
                security_mgr = SecurityManager()
                payload = security_mgr.verify_token(token, 'access')
                
                # Adicionar usuário ao contexto
                g.user_id = payload['user_id']
                g.user_type = payload['user_type']
                g.permissions = payload['permissions']
                
            except (IndexError, ValueError) as e:
                return jsonify({'error': 'invalid_token', 'message': str(e)}), 401
            
            return f(*args, **kwargs)
        return decorated_function
    
    @staticmethod
    def check_permissions(required_permission: str):
        """Decorator para verificar permissões específicas"""
        def decorator(f):
            @wraps(f)
            @SecurityMiddleware.require_authentication
            def decorated_function(*args, **kwargs):
                if required_permission not in g.permissions:
                    return jsonify({
                        'error': 'insufficient_permissions',
                        'required': required_permission,
                        'have': g.permissions
                    }), 403
                
                return f(*args, **kwargs)
            return decorated_function
        return decorator
    
    @staticmethod
    def validate_input(schema: dict):
        """Decorator para validação de entrada"""
        from jsonschema import validate, ValidationError
        
        def decorator(f):
            @wraps(f)
            def decorated_function(*args, **kwargs):
                data = request.get_json()
                
                if not data:
                    return jsonify({'error': 'no_data_provided'}), 400
                
                try:
                    # Validar contra schema
                    validate(instance=data, schema=schema)
                    
                    # Sanitizar entradas de texto
                    security_mgr = SecurityManager()
                    for key, value in data.items():
                        if isinstance(value, str):
                            data[key] = security_mgr.sanitize_input(value)
                    
                except ValidationError as e:
                    return jsonify({
                        'error': 'validation_error',
                        'message': e.message,
                        'path': list(e.path)
                    }), 400
                
                # Adicionar dados validados ao contexto
                g.validated_data = data
                
                return f(*args, **kwargs)
            return decorated_function
        return decorator
    
    @staticmethod
    def log_request():
        """Middleware para logging de requisições"""
        @wraps
        def middleware(f):
            def decorated_function(*args, **kwargs):
                start_time = time.time()
                
                # Registrar requisição
                security_mgr = SecurityManager()
                security_mgr.log_security_event(
                    event_type='request',
                    user_id=g.get('user_id'),
                    ip_address=request.remote_addr,
                    details={
                        'method': request.method,
                        'path': request.path,
                        'user_agent': request.headers.get('User-Agent'),
                        'content_type': request.content_type,
                        'content_length': request.content_length
                    }
                )
                
                try:
                    response = f(*args, **kwargs)
                    
                    # Calcular tempo de resposta
                    response_time = time.time() - start_time
                    
                    # Adicionar header com tempo de resposta
                    if isinstance(response, tuple):
                        response[0].headers['X-Response-Time'] = f'{response_time:.3f}s'
                    else:
                        response.headers['X-Response-Time'] = f'{response_time:.3f}s'
                    
                    return response
                    
                except Exception as e:
                    # Log de erro
                    security_mgr.log_security_event(
                        event_type='request_error',
                        user_id=g.get('user_id'),
                        ip_address=request.remote_addr,
                        details={
                            'method': request.method,
                            'path': request.path,
                            'error': str(e),
                            'error_type': type(e).__name__
                        }
                    )
                    raise
                    
            return decorated_function
        return middleware
```

### **9.4 Modelo de Permissões RBAC (Role-Based Access Control)**

#### **9.4.1 Estrutura de Permissões**
```sql
-- Tabela de papéis (roles)
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    is_system BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela de permissões
CREATE TABLE permissions (
    id SERIAL PRIMARY KEY,
    code VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    category VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela de atribuição de permissões a papéis
CREATE TABLE role_permissions (
    role_id INTEGER REFERENCES roles(id) ON DELETE CASCADE,
    permission_id INTEGER REFERENCES permissions(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (role_id, permission_id)
);

-- Tabela de atribuição de papéis a usuários
CREATE TABLE user_roles (
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    role_id INTEGER REFERENCES roles(id) ON DELETE CASCADE,
    assigned_by INTEGER REFERENCES users(id),
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    PRIMARY KEY (user_id, role_id)
);

-- Inserir papéis padrão
INSERT INTO roles (name, description, is_system) VALUES
('operator', 'Operador de máquina', true),
('supervisor', 'Supervisor de produção', true),
('maintenance', 'Técnico de manutenção', true),
('quality', 'Inspetor de qualidade', true),
('planner', 'Planejador de produção', true),
('admin', 'Administrador do sistema', true);

-- Inserir permissões padrão
INSERT INTO permissions (code, name, category) VALUES
-- Produção
('production.create', 'Criar apontamento', 'production'),
('production.read', 'Ver apontamentos', 'production'),
('production.update', 'Editar apontamento', 'production'),
('production.delete', 'Excluir apontamento', 'production'),
('production.finalize', 'Finalizar produção', 'production'),
('production.transfer', 'Transferir OP entre máquinas', 'production'),

-- Máquinas
('machine.view', 'Ver máquinas', 'machines'),
('machine.operate', 'Operar máquina', 'machines'),
('machine.stop', 'Parar máquina', 'machines'),
('machine.maintenance', 'Registrar manutenção', 'machines'),

-- Qualidade
('quality.inspect', 'Inspecionar produção', 'quality'),
('quality.approve', 'Aprovar produção', 'quality'),
('quality.reject', 'Rejeitar produção', 'quality'),
('quality.rework', 'Autorizar reprocesso', 'quality'),

-- Relatórios
('report.view', 'Ver relatórios', 'reports'),
('report.export', 'Exportar relatórios', 'reports'),
('report.custom', 'Criar relatórios personalizados', 'reports'),

-- Administração
('admin.users', 'Gerenciar usuários', 'administration'),
('admin.machines', 'Gerenciar máquinas', 'administration'),
('admin.products', 'Gerenciar produtos', 'administration'),
('admin.config', 'Gerenciar configurações', 'administration'),
('admin.backup', 'Gerenciar backups', 'administration'),

-- Sistema
('system.monitor', 'Monitorar sistema', 'system'),
('system.logs', 'Ver logs do sistema', 'system'),
('system.audit', 'Ver auditoria', 'system');

-- Atribuir permissões aos papéis
-- Operador
INSERT INTO role_permissions (role_id, permission_id)
SELECT r.id, p.id FROM roles r, permissions p
WHERE r.name = 'operator'
AND p.code IN (
    'production.create',
    'production.read',
    'production.update',
    'production.finalize',
    'machine.view',
    'machine.operate',
    'machine.stop',
    'report.view'
);

-- Supervisor
INSERT INTO role_permissions (role_id, permission_id)
SELECT r.id, p.id FROM roles r, permissions p
WHERE r.name = 'supervisor'
AND p.code IN (
    'production.create',
    'production.read',
    'production.update',
    'production.delete',
    'production.finalize',
    'production.transfer',
    'machine.view',
    'machine.operate',
    'machine.stop',
    'machine.maintenance',
    'quality.inspect',
    'quality.approve',
    'quality.reject',
    'report.view',
    'report.export'
);

-- Admin (todas as permissões)
INSERT INTO role_permissions (role_id, permission_id)
SELECT r.id, p.id FROM roles r, permissions p
WHERE r.name = 'admin';
```

#### **9.4.2 Gerenciador de Permissões em Python**
```python
# app/auth/permission_manager.py
from functools import wraps
from flask import g, jsonify

class PermissionManager:
    
    @staticmethod
    def has_permission(user_id: int, permission_code: str) -> bool:
        """Verificar se usuário tem permissão específica"""
        from models import User, Role, Permission
        
        user = User.query.get(user_id)
        if not user:
            return False
        
        # Se usuário é admin, tem todas as permissões
        if user.user_type == 'admin':
            return True
        
        # Verificar permissões via roles
        return db.session.query(Permission).join(
            Role.permissions
        ).join(
            User.roles
        ).filter(
            User.id == user_id,
            Permission.code == permission_code,
            User.is_active == True
        ).first() is not None
    
    @staticmethod
    def get_user_permissions(user_id: int) -> list:
        """Obter todas as permissões do usuário"""
        from models import User, Permission
        
        user = User.query.get(user_id)
        if not user or not user.is_active:
            return []
        
        # Admin tem todas as permissões
        if user.user_type == 'admin':
            permissions = Permission.query.all()
        else:
            permissions = Permission.query.join(
                Role.permissions
            ).join(
                User.roles
            ).filter(
                User.id == user_id
            ).all()
        
        return [p.code for p in permissions]
    
    @staticmethod
    def require_permission(permission_code: str):
        """Decorator para verificar permissão"""
        def decorator(f):
            @wraps(f)
            def decorated_function(*args, **kwargs):
                if not hasattr(g, 'user_id'):
                    return jsonify({'error': 'authentication_required'}), 401
                
                if not PermissionManager.has_permission(g.user_id, permission_code):
                    return jsonify({
                        'error': 'insufficient_permissions',
                        'required_permission': permission_code
                    }), 403
                
                return f(*args, **kwargs)
            return decorated_function
        return decorator
    
    @staticmethod
    def require_any_permission(*permission_codes):
        """Decorator para verificar qualquer uma das permissões"""
        def decorator(f):
            @wraps(f)
            def decorated_function(*args, **kwargs):
                if not hasattr(g, 'user_id'):
                    return jsonify({'error': 'authentication_required'}), 401
                
                for permission_code in permission_codes:
                    if PermissionManager.has_permission(g.user_id, permission_code):
                        return f(*args, **kwargs)
                
                return jsonify({
                    'error': 'insufficient_permissions',
                    'required_permissions': permission_codes
                }), 403
            return decorated_function
        return decorator
    
    @staticmethod
    def require_all_permissions(*permission_codes):
        """Decorator para verificar todas as permissões"""
        def decorator(f):
            @wraps(f)
            def decorated_function(*args, **kwargs):
                if not hasattr(g, 'user_id'):
                    return jsonify({'error': 'authentication_required'}), 401
                
                for permission_code in permission_codes:
                    if not PermissionManager.has_permission(g.user_id, permission_code):
                        return jsonify({
                            'error': 'insufficient_permissions',
                            'missing_permission': permission_code,
                            'required_permissions': permission_codes
                        }), 403
                
                return f(*args, **kwargs)
            return decorated_function
        return decorator
```

### **9.5 Monitoramento e Detecção de Intrusão**

#### **9.5.1 Sistema de Detecção de Comportamento Anômalo**
```python
# app/security/anomaly_detection.py
from datetime import datetime, timedelta
import statistics
from collections import defaultdict

class AnomalyDetector:
    
    def __init__(self):
        self.user_activity = defaultdict(list)
        self.ip_activity = defaultdict(list)
        self.alert_thresholds = {
            'failed_logins': 5,  # 5 tentativas em 10 minutos
            'rapid_requests': 100,  # 100 requisições em 1 minuto
            'unusual_hours': {
                'start': 6,  # 6h
                'end': 22    # 22h
            }
        }
    
    def record_login_attempt(self, username: str, ip_address: str, success: bool):
        """Registrar tentativa de login"""
        timestamp = datetime.utcnow()
        
        key = f"{username}:{ip_address}"
        
        self.user_activity[key].append({
            'timestamp': timestamp,
            'success': success,
            'type': 'login'
        })
        
        # Limpar registros antigos (mais de 24 horas)
        cutoff = timestamp - timedelta(hours=24)
        self.user_activity[key] = [
            record for record in self.user_activity[key]
            if record['timestamp'] > cutoff
        ]
        
        # Verificar anomalias
        anomalies = self.check_login_anomalies(username, ip_address)
        
        if anomalies:
            self.trigger_alerts(anomalies, username, ip_address)
    
    def check_login_anomalies(self, username: str, ip_address: str) -> list:
        """Verificar anomalias no login"""
        anomalies = []
        key = f"{username}:{ip_address}"
        
        # Verificar tentativas falhas recentes
        recent_failures = [
            record for record in self.user_activity[key]
            if not record['success']
            and record['timestamp'] > datetime.utcnow() - timedelta(minutes=10)
        ]
        
        if len(recent_failures) >= self.alert_thresholds['failed_logins']:
            anomalies.append({
                'type': 'failed_login_attempts',
                'count': len(recent_failures),
                'threshold': self.alert_thresholds['failed_logins']
            })
        
        # Verificar hora incomum
        current_hour = datetime.utcnow().hour
        if current_hour < self.alert_thresholds['unusual_hours']['start'] or \
           current_hour > self.alert_thresholds['unusual_hours']['end']:
            
            # Verificar se é padrão para este usuário
            user_logins = [
                record for record in self.user_activity[key]
                if record['success']
            ]
            
            if user_logins:
                login_hours = [record['timestamp'].hour for record in user_logins]
                avg_hour = statistics.mean(login_hours)
                
                if abs(current_hour - avg_hour) > 4:  # 4 horas de diferença
                    anomalies.append({
                        'type': 'unusual_login_time',
                        'current_hour': current_hour,
                        'average_hour': avg_hour
                    })
        
        return anomalies
    
    def record_request(self, user_id: int, ip_address: str, endpoint: str):
        """Registrar requisição HTTP"""
        timestamp = datetime.utcnow()
        
        self.ip_activity[ip_address].append({
            'timestamp': timestamp,
            'user_id': user_id,
            'endpoint': endpoint
        })
        
        # Limpar registros antigos
        cutoff = timestamp - timedelta(minutes=5)
        self.ip_activity[ip_address] = [
            record for record in self.ip_activity[ip_address]
            if record['timestamp'] > cutoff
        ]
        
        # Verificar requisições rápidas
        recent_requests = [
            record for record in self.ip_activity[ip_address]
            if record['timestamp'] > timestamp - timedelta(minutes=1)
        ]
        
        if len(recent_requests) >= self.alert_thresholds['rapid_requests']:
            anomalies = [{
                'type': 'rapid_requests',
                'count': len(recent_requests),
                'threshold': self.alert_thresholds['rapid_requests']
            }]
            
            self.trigger_alerts(anomalies, user_id, ip_address)
    
    def trigger_alerts(self, anomalies: list, identifier: str, ip_address: str):
        """Disparar alertas de segurança"""
        from app import db
        from models import SecurityAlert
        
        for anomaly in anomalies:
            alert = SecurityAlert(
                alert_type=anomaly['type'],
                severity='high' if anomaly['type'] == 'rapid_requests' else 'medium',
                identifier=identifier,
                ip_address=ip_address,
                details=anomaly,
                created_at=datetime.utcnow()
            )
            
            db.session.add(alert)
            
            # Log adicional
            print(f"[SECURITY ALERT] {anomaly['type']} from {identifier} ({ip_address})")
        
        db.session.commit()
        
        # Notificar administradores (em produção, seria email/slack/etc)
        if any(a['type'] == 'rapid_requests' for a in anomalies):
            self.notify_administrators(anomalies, identifier, ip_address)
    
    def notify_administrators(self, anomalies: list, identifier: str, ip_address: str):
        """Notificar administradores sobre alertas críticos"""
        # Em produção, implementar notificação por email, Slack, etc.
        subject = f"[CRÍTICO] Possível ataque detectado - {identifier}"
        message = f"""
        Alerta de segurança crítico detectado:
        
        Identificador: {identifier}
        IP: {ip_address}
        Tipo: {anomalies[0]['type']}
        Detalhes: {anomalies[0]}
        Horário: {datetime.utcnow().isoformat()}
        
        Ação recomendada:
        1. Verificar logs do sistema
        2. Bloquear IP se necessário
        3. Notificar equipe de segurança
        """
        
        # Aqui iria o código para enviar email
        # send_email_to_admins(subject, message)
        
        print(f"[CRITICAL ALERT] {subject}")
```

#### **9.5.2 Configuração de Logs de Segurança**
```python
# app/logging/security_logger.py
import logging
import json
from datetime import datetime
from pythonjsonlogger import jsonlogger

class SecurityLogger:
    
    def __init__(self):
        # Configurar logger JSON
        self.logger = logging.getLogger('security')
        self.logger.setLevel(logging.INFO)
        
        # Handler para arquivo
        file_handler = logging.FileHandler('/opt/productionpointer/data/logs/security.json')
        
        # Formatter JSON
        formatter = jsonlogger.JsonFormatter(
            '%(timestamp)s %(level)s %(name)s %(message)s %(module)s %(funcName)s'
        )
        file_handler.setFormatter(formatter)
        
        self.logger.addHandler(file_handler)
    
    def log_security_event(self, event_type: str, **kwargs):
        """Registrar evento de segurança"""
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'event_type': event_type,
            'level': 'INFO',
            'name': 'security',
            **kwargs
        }
        
        # Determinar nível de log baseado no tipo de evento
        if event_type in ['login_failed', 'brute_force', 'sql_injection']:
            log_data['level'] = 'WARNING'
        elif event_type in ['access_denied', 'rate_limit_exceeded']:
            log_data['level'] = 'ERROR'
        
        self.logger.log(
            getattr(logging, log_data['level']),
            json.dumps(log_data)
        )
        
        # Também registrar no banco de dados
        self.save_to_database(event_type, **kwargs)
    
    def save_to_database(self, event_type: str, **kwargs):
        """Salvar evento no banco de dados"""
        from models import SecurityEvent
        
        event = SecurityEvent(
            event_type=event_type,
            user_id=kwargs.get('user_id'),
            ip_address=kwargs.get('ip_address'),
            user_agent=kwargs.get('user_agent'),
            details=kwargs,
            created_at=datetime.utcnow()
        )
        
        from app import db
        db.session.add(event)
        db.session.commit()

# Configuração do logger principal
def setup_logging():
    """Configurar sistema de logging completo"""
    
    # Logger da aplicação
    app_logger = logging.getLogger('production_pointer')
    app_logger.setLevel(logging.INFO)
    
    # Handler para arquivo
    file_handler = logging.FileHandler('/opt/productionpointer/data/logs/app.log')
    file_handler.setLevel(logging.INFO)
    
    # Formatter detalhado
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    file_handler.setFormatter(formatter)
    
    app_logger.addHandler(file_handler)
    
    # Handler para console (apenas em desenvolvimento)
    if os.getenv('FLASK_ENV') == 'development':
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.DEBUG)
        console_handler.setFormatter(formatter)
        app_logger.addHandler(console_handler)
    
    # Logger de segurança
    security_logger = SecurityLogger()
    
    # Logger de auditoria do banco de dados
    audit_logger = logging.getLogger('sqlalchemy.engine')
    audit_logger.setLevel(logging.WARNING)
    
    return app_logger, security_logger
```

---

## **10. BACKUP E RECUPERAÇÃO DE DESASTRES**

### **10.1 Estratégia de Backup 3-2-1**

```
┌─────────────────────────────────────────────────────────────┐
│                  ESTRATÉGIA DE BACKUP 3-2-1                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  3 CÓPIAS dos dados                                        │
│    • Original no servidor de produção                       │
│    • Backup local no mesmo datacenter                       │
│    • Backup remoto em localização geográfica diferente      │
│                                                             │
│  2 TIPOS DIFERENTES de mídia                               │
│    • SSD/HDD para backup local rápido                      │
│    • Fita LTO ou cloud storage para backup remoto          │
│                                                             │
│  1 CÓPIA OFFSITE                                            │
│    • Backup em localização fisicamente separada            │
│    • Proteção contra desastres naturais, incêndios, etc.   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### **10.2 Scripts de Backup Completos**

#### **10.2.1 Script de Backup Diário**
```bash
#!/bin/bash
# /opt/productionpointer/scripts/backup_daily.sh

# Configurações
BACKUP_DIR="/opt/productionpointer/data/backups"
LOG_FILE="/opt/productionpointer/data/logs/backup.log"
RETENTION_DAYS=30
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="production_pointer_${DATE}"

# Função para logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Função para verificar espaço em disco
check_disk_space() {
    local required_gb=5  # 5GB mínimo necessário
    local available_gb=$(df -BG "$BACKUP_DIR" | awk 'NR==2 {print $4}' | sed 's/G//')
    
    if [ "$available_gb" -lt "$required_gb" ]; then
        log "ERRO: Espaço insuficiente. Disponível: ${available_gb}GB, Necessário: ${required_gb}GB"
        return 1
    fi
    return 0
}

# Função para enviar notificação
send_notification() {
    local subject="$1"
    local message="$2"
    
    # Enviar email (ajustar para seu sistema de email)
    echo "$message" | mail -s "$subject" admin@empresa.com.br
    
    # Log da notificação
    log "Notificação enviada: $subject"
}

# Iniciar backup
log "=== INICIANDO BACKUP DIÁRIO ==="

# 1. Criar diretório de backup
BACKUP_PATH="${BACKUP_DIR}/daily/${BACKUP_NAME}"
mkdir -p "$BACKUP_PATH"
log "Diretório criado: $BACKUP_PATH"

# 2. Verificar espaço em disco
if ! check_disk_space; then
    send_notification "ERRO DE BACKUP - Espaço Insuficiente" "Verifique o espaço em disco no servidor de produção."
    exit 1
fi

# 3. Backup do PostgreSQL
log "Iniciando backup do PostgreSQL..."
PG_BACKUP_FILE="${BACKUP_PATH}/postgres_backup.sql"
PG_DUMP_OPTIONS="--no-password --format=custom --verbose --blobs"
sudo -u postgres pg_dump $PG_DUMP_OPTIONS production_pointer > "$PG_BACKUP_FILE"

if [ $? -eq 0 ]; then
    PG_SIZE=$(du -h "$PG_BACKUP_FILE" | cut -f1)
    log "Backup PostgreSQL concluído: $PG_BACKUP_FILE ($PG_SIZE)"
else
    log "ERRO: Falha no backup do PostgreSQL"
    send_notification "ERRO DE BACKUP - PostgreSQL" "Falha no backup do banco de dados PostgreSQL."
    exit 1
fi

# 4. Backup do Redis
log "Iniciando backup do Redis..."
REDIS_BACKUP_FILE="${BACKUP_PATH}/redis_backup.rdb"
REDIS_TEMP_FILE="/tmp/redis_dump_${DATE}.rdb"

# Salvar dados do Redis
redis-cli -a "$REDIS_PASSWORD" SAVE
cp /var/lib/redis/dump.rdb "$REDIS_TEMP_FILE"
cp "$REDIS_TEMP_FILE" "$REDIS_BACKUP_FILE"
rm "$REDIS_TEMP_FILE"

if [ -f "$REDIS_BACKUP_FILE" ]; then
    REDIS_SIZE=$(du -h "$REDIS_BACKUP_FILE" | cut -f1)
    log "Backup Redis concluído: $REDIS_BACKUP_FILE ($REDIS_SIZE)"
else
    log "ERRO: Falha no backup do Redis"
    send_notification "ERRO DE BACKUP - Redis" "Falha no backup do Redis."
fi

# 5. Backup dos arquivos da aplicação
log "Iniciando backup dos arquivos da aplicação..."
APP_BACKUP_FILE="${BACKUP_PATH}/app_backup.tar.gz"

# Lista de diretórios importantes
tar -czf "$APP_BACKUP_FILE" \
    --exclude="*.pyc" \
    --exclude="__pycache__" \
    --exclude=".git" \
    --exclude="node_modules" \
    /opt/productionpointer/app \
    /opt/productionpointer/venv \
    /opt/productionpointer/config \
    /opt/productionpointer/scripts

if [ $? -eq 0 ]; then
    APP_SIZE=$(du -h "$APP_BACKUP_FILE" | cut -f1)
    log "Backup da aplicação concluído: $APP_BACKUP_FILE ($APP_SIZE)"
else
    log "ERRO: Falha no backup dos arquivos da aplicação"
    send_notification "ERRO DE BACKUP - Arquivos da Aplicação" "Falha no backup dos arquivos da aplicação."
fi

# 6. Backup dos uploads
log "Iniciando backup dos uploads..."
UPLOADS_BACKUP_FILE="${BACKUP_PATH}/uploads_backup.tar.gz"

if [ -d "/opt/productionpointer/data/uploads" ]; then
    tar -czf "$UPLOADS_BACKUP_FILE" /opt/productionpointer/data/uploads
    
    if [ $? -eq 0 ]; then
        UPLOADS_SIZE=$(du -h "$UPLOADS_BACKUP_FILE" | cut -f1)
        log "Backup dos uploads concluído: $UPLOADS_BACKUP_FILE ($UPLOADS_SIZE)"
    else
        log "ERRO: Falha no backup dos uploads"
    fi
else
    log "AVISO: Diretório de uploads não encontrado"
fi

# 7. Backup das configurações do sistema
log "Iniciando backup das configurações do sistema..."
CONFIG_BACKUP_FILE="${BACKUP_PATH}/system_config.tar.gz"

tar -czf "$CONFIG_BACKUP_FILE" \
    /etc/nginx/sites-available/productionpointer \
    /etc/supervisor/conf.d/productionpointer.conf \
    /etc/postgresql/15/main/postgresql.conf \
    /etc/postgresql/15/main/pg_hba.conf \
    /etc/redis/redis.conf

if [ $? -eq 0 ]; then
    CONFIG_SIZE=$(du -h "$CONFIG_BACKUP_FILE" | cut -f1)
    log "Backup das configurações concluído: $CONFIG_BACKUP_FILE ($CONFIG_SIZE)"
else
    log "ERRO: Falha no backup das configurações"
fi

# 8. Criar arquivo de manifesto
log "Criando arquivo de manifesto..."
MANIFEST_FILE="${BACKUP_PATH}/manifest.json"

cat > "$MANIFEST_FILE" << EOF
{
    "backup_name": "${BACKUP_NAME}",
    "timestamp": "$(date -Iseconds)",
    "components": {
        "postgresql": {
            "file": "$(basename $PG_BACKUP_FILE)",
            "size": "$PG_SIZE",
            "database": "production_pointer"
        },
        "redis": {
            "file": "$(basename $REDIS_BACKUP_FILE)",
            "size": "$REDIS_SIZE"
        },
        "application": {
            "file": "$(basename $APP_BACKUP_FILE)",
            "size": "$APP_SIZE"
        },
        "uploads": {
            "file": "$(basename $UPLOADS_BACKUP_FILE)",
            "size": "$UPLOADS_SIZE"
        },
        "config": {
            "file": "$(basename $CONFIG_BACKUP_FILE)",
            "size": "$CONFIG_SIZE"
        }
    },
    "system_info": {
        "hostname": "$(hostname)",
        "os": "$(lsb_release -ds)",
        "disk_usage": "$(df -h /opt | tail -1)",
        "memory_usage": "$(free -h | awk '/^Mem:/ {print $3 \"/\" $2}')"
    }
}
EOF

# 9. Calcular hash de verificação
log "Calculando hashes de verificação..."
MD5_FILE="${BACKUP_PATH}/checksums.md5"

cd "$BACKUP_PATH"
md5sum *.gz *.sql *.rdb manifest.json > "$MD5_FILE"

# 10. Limpar backups antigos
log "Limpando backups antigos (mais de ${RETENTION_DAYS} dias)..."
find "${BACKUP_DIR}/daily" -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;

# 11. Estatísticas finais
TOTAL_SIZE=$(du -sh "$BACKUP_PATH" | cut -f1)
BACKUP_COUNT=$(find "${BACKUP_DIR}/daily" -type d | wc -l)

log "Backup concluído com sucesso!"
log "Tamanho total: $TOTAL_SIZE"
log "Total de backups mantidos: $((BACKUP_COUNT - 1))"
log "Backup salvo em: $BACKUP_PATH"
log "=== BACKUP FINALIZADO ==="

# 12. Notificação de sucesso
send_notification "Backup Diário Concluído" "Backup do Production Pointer concluído com sucesso.\nTamanho: $TOTAL_SIZE\nLocal: $BACKUP_PATH"

exit 0
```

#### **10.2.2 Script de Backup para Nuvem (S3/Wasabi)**
```bash
#!/bin/bash
# /opt/productionpointer/scripts/backup_to_cloud.sh

# Configurações
BACKUP_DIR="/opt/productionpointer/data/backups"
CLOUD_DIR="s3://empresa-backups/production-pointer/"
LOG_FILE="/opt/productionpointer/data/logs/cloud_backup.log"
RETENTION_DAYS=90

# Credenciais (deveriam estar em variáveis de ambiente)
export AWS_ACCESS_KEY_ID="SUA_ACCESS_KEY"
export AWS_SECRET_ACCESS_KEY="SUA_SECRET_KEY"
export AWS_DEFAULT_REGION="us-east-1"

# Função para logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Função para compactar e criptografar
encrypt_backup() {
    local input_file="$1"
    local output_file="${input_file}.gpg"
    local passphrase="SUA_SENHA_FORTE_AQUI"  # Usar variável de ambiente na produção
    
    log "Criptografando: $(basename $input_file)"
    
    gpg --batch --yes --passphrase "$passphrase" \
        --cipher-algo AES256 \
        --symmetric \
        --output "$output_file" \
        "$input_file"
    
    if [ $? -eq 0 ]; then
        log "Criptografia concluída: $(basename $output_file)"
        rm "$input_file"  # Remover arquivo não criptografado
        echo "$output_file"
    else
        log "ERRO: Falha na criptografia de $(basename $input_file)"
        echo "$input_file"
    fi
}

# Função para enviar para nuvem
upload_to_cloud() {
    local file_path="$1"
    local file_name=$(basename "$file_path")
    
    log "Enviando para nuvem: $file_name"
    
    # Usar AWS CLI para enviar para S3
    aws s3 cp "$file_path" "${CLOUD_DIR}daily/${file_name}" \
        --storage-class STANDARD_IA \
        --metadata "backup-date=$(date -Iseconds)"
    
    if [ $? -eq 0 ]; then
        log "Upload concluído: $file_name"
        return 0
    else
        log "ERRO: Falha no upload de $file_name"
        return 1
    fi
}

# Iniciar backup na nuvem
log "=== INICIANDO BACKUP PARA NUVEM ==="

# 1. Encontrar backup mais recente
LATEST_BACKUP=$(find "${BACKUP_DIR}/daily" -type d -name "production_pointer_*" | sort -r | head -1)

if [ -z "$LATEST_BACKUP" ]; then
    log "ERRO: Nenhum backup local encontrado"
    exit 1
fi

log "Backup local selecionado: $(basename $LATEST_BACKUP)"

# 2. Processar cada arquivo do backup
for file in "$LATEST_BACKUP"/*; do
    if [ -f "$file" ]; then
        # Criptografar arquivo
        encrypted_file=$(encrypt_backup "$file")
        
        # Enviar para nuvem
        if upload_to_cloud "$encrypted_file"; then
            # Remover arquivo local criptografado após upload
            rm "$encrypted_file"
        fi
    fi
done

# 3. Manter apenas últimos X backups na nuvem
log "Limpando backups antigos na nuvem..."
aws s3 ls "${CLOUD_DIR}daily/" | awk '{print $4}' | sort -r | tail -n +$((RETENTION_DAYS + 1)) | while read file; do
    log "Removendo backup antigo: $file"
    aws s3 rm "${CLOUD_DIR}daily/$file"
done

# 4. Backup semanal completo (domingo)
if [ $(date +%u) -eq 7 ]; then  # Domingo = 7
    log "Criando backup semanal completo..."
    
    # Copiar backup diário para pasta semanal
    WEEKLY_DATE=$(date +%Y%m%d)
    aws s3 sync "${CLOUD_DIR}daily/" "${CLOUD_DIR}weekly/${WEEKLY_DATE}/"
    
    # Limpar backups semanais antigos (mantém 12 semanas)
    aws s3 ls "${CLOUD_DIR}weekly/" | awk '{print $2}' | sort -r | tail -n +13 | while read dir; do
        log "Removendo backup semanal antigo: $dir"
        aws s3 rm --recursive "${CLOUD_DIR}weekly/$dir"
    done
fi

# 5. Backup mensal (primeiro dia do mês)
if [ $(date +%d) -eq 1 ]; then
    log "Criando backup mensal..."
    
    MONTHLY_DATE=$(date +%Y%m)
    aws s3 sync "${CLOUD_DIR}daily/" "${CLOUD_DIR}monthly/${MONTHLY_DATE}/"
    
    # Limpar backups mensais antigos (mantém 12 meses)
    aws s3 ls "${CLOUD_DIR}monthly/" | awk '{print $2}' | sort -r | tail -n +13 | while read dir; do
        log "Removendo backup mensal antigo: $dir"
        aws s3 rm --recursive "${CLOUD_DIR}monthly/$dir"
    done
fi

log "=== BACKUP PARA NUVEM CONCLUÍDO ==="

# Enviar notificação
echo "Backup na nuvem concluído em $(date)" | mail -s "Backup Nuvem OK" admin@empresa.com.br

exit 0
```

#### **10.2.3 Script de Recuperação de Desastre**
```bash
#!/bin/bash
# /opt/productionpointer/scripts/disaster_recovery.sh

# Configurações
RESTORE_DIR="/opt/productionpointer/restore"
BACKUP_SOURCE="/opt/productionpointer/data/backups/daily"
LOG_FILE="/opt/productionpointer/data/logs/recovery.log"
TEMP_DIR="/tmp/recovery_$(date +%Y%m%d_%H%M%S)"

# Função para logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Função para escolher backup
choose_backup() {
    local backups=($(find "$BACKUP_SOURCE" -type d -name "production_pointer_*" | sort -r))
    
    if [ ${#backups[@]} -eq 0 ]; then
        log "ERRO: Nenhum backup disponível"
        exit 1
    fi
    
    echo "Backups disponíveis:"
    for i in "${!backups[@]}"; do
        echo "[$i] $(basename ${backups[$i]})"
    done
    
    read -p "Selecione o número do backup para restaurar: " choice
    
    if [[ "$choice" =~ ^[0-9]+$ ]] && [ "$choice" -lt "${#backups[@]}" ]; then
        echo "${backups[$choice]}"
    else
        log "ERRO: Seleção inválida"
        exit 1
    fi
}

# Função para parar serviços
stop_services() {
    log "Parando serviços..."
    
    sudo systemctl stop nginx
    sudo systemctl stop productionpointer
    sudo systemctl stop celery
    sudo systemctl stop celerybeat
    sudo systemctl stop redis-server
    sudo systemctl stop postgresql
    
    log "Serviços parados"
}

# Função para iniciar serviços
start_services() {
    log "Iniciando serviços..."
    
    sudo systemctl start postgresql
    sudo systemctl start redis-server
    sudo systemctl start celery
    sudo systemctl start celerybeat
    sudo systemctl start productionpointer
    sudo systemctl start nginx
    
    log "Serviços iniciados"
}

# Função para verificar integridade
verify_backup() {
    local backup_path="$1"
    
    log "Verificando integridade do backup..."
    
    # Verificar arquivo de checksum
    if [ ! -f "$backup_path/checksums.md5" ]; then
        log "AVISO: Arquivo de checksum não encontrado"
        return 1
    fi
    
    # Verificar checksums
    cd "$backup_path"
    if md5sum -c checksums.md5 > /dev/null 2>&1; then
        log "Integridade do backup verificada com sucesso"
        return 0
    else
        log "ERRO: Checksum inválido. Backup pode estar corrompido."
        return 1
    fi
}

# Função para restaurar PostgreSQL
restore_postgresql() {
    local backup_file="$1/postgres_backup.sql"
    
    if [ ! -f "$backup_file" ]; then
        log "ERRO: Arquivo de backup PostgreSQL não encontrado"
        return 1
    fi
    
    log "Restaurando PostgreSQL..."
    
    # Parar conexões ativas
    sudo -u postgres psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'production_pointer';"
    
    # Dropar e recriar database
    sudo -u postgres psql -c "DROP DATABASE IF EXISTS production_pointer;"
    sudo -u postgres psql -c "CREATE DATABASE production_pointer;"
    
    # Restaurar backup
    sudo -u postgres pg_restore --no-password --verbose --clean \
        --dbname=production_pointer "$backup_file"
    
    if [ $? -eq 0 ]; then
        log "PostgreSQL restaurado com sucesso"
        return 0
    else
        log "ERRO: Falha na restauração do PostgreSQL"
        return 1
    fi
}

# Função para restaurar Redis
restore_redis() {
    local backup_file="$1/redis_backup.rdb"
    
    if [ ! -f "$backup_file" ]; then
        log "AVISO: Arquivo de backup Redis não encontrado"
        return 0  # Não é crítico
    fi
    
    log "Restaurando Redis..."
    
    # Parar Redis
    sudo systemctl stop redis-server
    
    # Copiar arquivo RDB
    cp "$backup_file" /var/lib/redis/dump.rdb
    chown redis:redis /var/lib/redis/dump.rdb
    
    # Iniciar Redis
    sudo systemctl start redis-server
    
    log "Redis restaurado com sucesso"
    return 0
}

# Função para restaurar arquivos da aplicação
restore_application() {
    local backup_file="$1/app_backup.tar.gz"
    
    if [ ! -f "$backup_file" ]; then
        log "ERRO: Arquivo de backup da aplicação não encontrado"
        return 1
    fi
    
    log "Restaurando arquivos da aplicação..."
    
    # Criar backup dos arquivos atuais
    mkdir -p "$TEMP_DIR/old_app"
    mv /opt/productionpointer/app "$TEMP_DIR/old_app/app" 2>/dev/null || true
    mv /opt/productionpointer/venv "$TEMP_DIR/old_app/venv" 2>/dev/null || true
    
    # Extrair backup
    tar -xzf "$backup_file" -C /
    
    # Ajustar permissões
    chown -R production:production /opt/productionpointer/app
    chmod -R 750 /opt/productionpointer/app
    
    log "Aplicação restaurada com sucesso"
    return 0
}

# Função para restaurar configurações
restore_config() {
    local backup_file="$1/system_config.tar.gz"
    
    if [ ! -f "$backup_file" ]; then
        log "AVISO: Arquivo de backup de configurações não encontrado"
        return 0
    fi
    
    log "Restaurando configurações do sistema..."
    
    # Extrair configurações
    tar -xzf "$backup_file" -C /
    
    # Recarregar serviços
    sudo systemctl daemon-reload
    
    log "Configurações restauradas com sucesso"
    return 0
}

# Função para restaurar uploads
restore_uploads() {
    local backup_file="$1/uploads_backup.tar.gz"
    
    if [ ! -f "$backup_file" ]; then
        log "AVISO: Arquivo de backup de uploads não encontrado"
        return 0
    fi
    
    log "Restaurando uploads..."
    
    # Criar diretório se não existir
    mkdir -p /opt/productionpointer/data/uploads
    
    # Extrair uploads
    tar -xzf "$backup_file" -C /
    
    # Ajustar permissões
    chown -R production:production /opt/productionpointer/data/uploads
    
    log "Uploads restaurados com sucesso"
    return 0
}

# Função para teste de recuperação
test_recovery() {
    log "Realizando teste de recuperação..."
    
    # Testar conexão com PostgreSQL
    if sudo -u postgres psql -c "\c production_pointer" > /dev/null 2>&1; then
        log "✓ PostgreSQL conectado"
    else
        log "✗ ERRO: PostgreSQL não conectado"
        return 1
    fi
    
    # Testar Redis
    if redis-cli -a "$REDIS_PASSWORD" PING > /dev/null 2>&1; then
        log "✓ Redis conectado"
    else
        log "✗ ERRO: Redis não conectado"
        return 1
    fi
    
    # Testar aplicação
    if curl -f http://localhost:8000/health > /dev/null 2>&1; then
        log "✓ Aplicação respondendo"
    else
        log "✗ ERRO: Aplicação não respondendo"
        return 1
    fi
    
    log "✓ Teste de recuperação concluído com sucesso"
    return 0
}

# Função principal
main() {
    log "=== INICIANDO RECUPERAÇÃO DE DESASTRE ==="
    
    # 1. Escolher backup
    BACKUP_PATH=$(choose_backup)
    log "Backup selecionado: $BACKUP_PATH"
    
    # 2. Verificar integridade
    if ! verify_backup "$BACKUP_PATH"; then
        read -p "Backup pode estar corrompido. Continuar? (s/N): " confirm
        if [[ ! "$confirm" =~ ^[Ss]$ ]]; then
            log "Recuperação cancelada pelo usuário"
            exit 1
        fi
    fi
    
    # 3. Confirmar ação
    echo "=============================================="
    echo "ATENÇÃO: Esta operação irá:"
    echo "1. Parar todos os serviços"
    echo "2. Restaurar banco de dados (dados atuais serão perdidos)"
    echo "3. Restaurar arquivos da aplicação"
    echo "4. Restaurar configurações"
    echo "=============================================="
    
    read -p "Confirmar recuperação? (Digite 'CONFIRMAR'): " confirm
    
    if [ "$confirm" != "CONFIRMAR" ]; then
        log "Recuperação cancelada pelo usuário"
        exit 0
    fi
    
    # 4. Parar serviços
    stop_services
    
    # 5. Criar diretório temporário
    mkdir -p "$TEMP_DIR"
    
    # 6. Restaurar componentes
    restore_postgresql "$BACKUP_PATH"
    restore_redis "$BACKUP_PATH"
    restore_application "$BACKUP_PATH"
    restore_config "$BACKUP_PATH"
    restore_uploads "$BACKUP_PATH"
    
    # 7. Iniciar serviços
    start_services
    
    # 8. Testar recuperação
    if test_recovery; then
        log "✓ Recuperação concluída com sucesso!"
        
        # Limpar diretório temporário
        rm -rf "$TEMP_DIR"
        
        # Registrar evento
        echo "Recuperação de desastre concluída em $(date)" | \
            mail -s "RECUPERAÇÃO CONCLUÍDA" admin@empresa.com.br
        
    else
        log "✗ ERRO: Teste de recuperação falhou"
        
        # Oferecer rollback
        read -p "Falha na recuperação. Reverter para estado anterior? (s/N): " rollback
        
        if [[ "$rollback" =~ ^[Ss]$ ]]; then
            log "Iniciando rollback..."
            
            # Restaurar estado anterior
            if [ -d "$TEMP_DIR/old_app" ]; then
                rm -rf /opt/productionpointer/app
                rm -rf /opt/productionpointer/venv
                mv "$TEMP_DIR/old_app"/* /opt/productionpointer/
            fi
            
            start_services
            log "Rollback concluído"
        fi
        
        exit 1
    fi
    
    log "=== RECUPERAÇÃO FINALIZADA ==="
}

# Executar função principal
main
```

### **10.3 Procedimentos de Recuperação de Desastres**

#### **10.3.1 Cenários de Recuperação**

```
CENÁRIO 1: Falha de Hardware do Servidor
----------------------------------------
Sintomas:
• Servidor não responde
• LEDs de erro no hardware
• Alerta de temperatura

Procedimento:
1. Diagnosticar hardware (se possível)
2. Se irreparável rápido, iniciar recuperação
3. Usar backup mais recente
4. Restaurar em servidor substituto
5. Atualizar DNS/IP se necessário
6. Testar completamente
7. Documentar incidente

Tempo Estimado: 2-4 horas
```

```
CENÁRIO 2: Corrupção de Banco de Dados
--------------------------------------
Sintomas:
• Aplicação retorna erros de banco
• Consultas falham
• Logs do PostgreSQL mostram erros

Procedimento:
1. Parar aplicação
2. Fazer backup atual (se possível)
3. Identificar ponto de corrupção
4. Restaurar backup mais recente
5. Aplicar logs de transação (WAL)
6. Verificar integridade
7. Reativar aplicação

Tempo Estimado: 1-2 horas
```

```
CENÁRIO 3: Ataque Ransomware
----------------------------
Sintomas:
• Arquivos criptografados
• Nota de resgate
• Comportamento estranho do sistema

Procedimento:
1. Isolar servidor da rede
2. Não pagar resgate
3. Identificar versão do ransomware
4. Verificar se há decryptor disponível
5. Restaurar de backup limpo
6. Reinstalar sistema operacional
7. Aplicar patches de segurança
8. Restaurar dados

Tempo Estimado: 4-8 horas
```

```
CENÁRIO 4: Exclusão Acidental de Dados
--------------------------------------
Sintomas:
• Dados desaparecidos
• Usuário reporta erro
• Logs mostram DELETE acidental

Procedimento:
1. Parar aplicação para evitar mais danos
2. Verificar backups disponíveis
3. Restaurar apenas tabelas afetadas
4. Usar PITR (Point-in-Time Recovery)
5. Validar dados recuperados
6. Reativar aplicação
7. Implementar medidas preventivas

Tempo Estimado: 1-3 horas
```

#### **10.3.2 Checklist de Recuperação**
```bash
#!/bin/bash
# recovery_checklist.sh

echo "=== CHECKLIST DE RECUPERAÇÃO DE DESASTRES ==="
echo ""
echo "ANTES DA RECUPERAÇÃO:"
echo "[ ] 1. Notificar stakeholders"
echo "[ ] 2. Documentar hora do incidente"
echo "[ ] 3. Identificar causa raiz"
echo "[ ] 4. Escolher ponto de recuperação"
echo "[ ] 5. Verificar disponibilidade de backup"
echo "[ ] 6. Alocar recursos necessários"
echo "[ ] 7. Comunicar tempo estimado de recuperação"
echo ""
echo "DURANTE A RECUPERAÇÃO:"
echo "[ ] 1. Parar serviços de produção"
echo "[ ] 2. Fazer backup do estado atual"
echo "[ ] 3. Verificar integridade do backup"
echo "[ ] 4. Restaurar banco de dados"
echo "[ ] 5. Restaurar arquivos da aplicação"
echo "[ ] 6. Restaurar configurações"
echo "[ ] 7. Restaurar dados de sessão (se aplicável)"
echo "[ ] 8. Verificar permissões de arquivos"
echo ""
echo "APÓS A RECUPERAÇÃO:"
echo "[ ] 1. Iniciar serviços"
echo "[ ] 2. Realizar testes funcionais"
echo "[ ] 3. Verificar integridade de dados"
echo "[ ] 4. Testar desempenho"
echo "[ ] 5. Validar com usuários-chave"
echo "[ ] 6. Monitorar por anomalias"
echo "[ ] 7. Documentar recuperação"
echo "[ ] 8. Atualizar plano de recuperação"
echo "[ ] 9. Agendar análise pós-incidente"
echo ""
echo "TESTES A REALIZAR:"
echo "[ ] Login de usuários"
echo "[ ] Apontamento de produção"
echo "[ ] Geração de relatórios"
echo "[ ] Sincronização com ERP"
echo "[ ] Performance do sistema"
echo "[ ] Funcionalidades críticas"
echo ""
read -p "Pressione Enter para continuar..."
```

### **10.4 Backup em Nível de Bloco (LVM Snapshots)**

#### **10.4.1 Configurar LVM para Snapshots**
```bash
#!/bin/bash
# setup_lvm_snapshots.sh

# 1. Verificar configuração atual de discos
lsblk
sudo fdisk -l

# 2. Criar volume físico LVM (se não existir)
# Supondo que /dev/sdb é o disco adicional
sudo pvcreate /dev/sdb
sudo vgcreate vg_backup /dev/sdb

# 3. Criar volume lógico para backups
sudo lvcreate -L 100G -n lv_backup vg_backup

# 4. Formatar e montar
sudo mkfs.ext4 /dev/vg_backup/lv_backup
sudo mkdir -p /mnt/backup
sudo mount /dev/vg_backup/lv_backup /mnt/backup

# 5. Adicionar ao fstab
echo "/dev/vg_backup/lv_backup /mnt/backup ext4 defaults 0 2" | sudo tee -a /etc/fstab

# 6. Criar script de snapshot
sudo tee /usr/local/bin/create_snapshot.sh << 'EOF'
#!/bin/bash

# Configurações
VG_NAME="vg_production"  # Grupo de volume onde está o sistema
LV_NAME="lv_production"  # Volume lógico do sistema
SNAPSHOT_SIZE="10G"      # Tamanho do snapshot
SNAPSHOT_NAME="${LV_NAME}_snap_$(date +%Y%m%d_%H%M%S)"
MOUNT_POINT="/mnt/snapshot"
BACKUP_DIR="/mnt/backup"

# Função para logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Verificar se snapshot já existe
if lvs | grep -q "${LV_NAME}_snap"; then
    log "ERRO: Snapshot já existe. Remova antes de criar novo."
    exit 1
fi

# 1. Criar snapshot
log "Criando snapshot: $SNAPSHOT_NAME"
lvcreate -L $SNAPSHOT_SIZE -s -n $SNAPSHOT_NAME /dev/$VG_NAME/$LV_NAME

if [ $? -ne 0 ]; then
    log "ERRO: Falha ao criar snapshot"
    exit 1
fi

# 2. Montar snapshot
mkdir -p $MOUNT_POINT
mount -o ro /dev/$VG_NAME/$SNAPSHOT_NAME $MOUNT_POINT

if [ $? -ne 0 ]; then
    log "ERRO: Falha ao montar snapshot"
    lvremove -f /dev/$VG_NAME/$SNAPSHOT_NAME
    exit 1
fi

# 3. Criar backup do snapshot
log "Criando backup do snapshot..."
BACKUP_FILE="${BACKUP_DIR}/snapshot_$(date +%Y%m%d_%H%M%S).tar.gz"

tar -czf "$BACKUP_FILE" \
    --exclude="/proc" \
    --exclude="/sys" \
    --exclude="/dev" \
    --exclude="/run" \
    --exclude="/tmp" \
    --exclude="/mnt" \
    --exclude="/media" \
    -C $MOUNT_POINT .

if [ $? -eq 0 ]; then
    BACKUP_SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
    log "Backup criado: $BACKUP_FILE ($BACKUP_SIZE)"
else
    log "ERRO: Falha ao criar backup"
fi

# 4. Desmontar e remover snapshot
log "Limpando snapshot..."
umount $MOUNT_POINT
lvremove -f /dev/$VG_NAME/$SNAPSHOT_NAME
rmdir $MOUNT_POINT

log "Snapshot concluído com sucesso"
EOF

sudo chmod +x /usr/local/bin/create_snapshot.sh

# 7. Agendar snapshot diário
sudo tee /etc/cron.daily/snapshot_backup << 'EOF'
#!/bin/bash

/usr/local/bin/create_snapshot.sh >> /var/log/snapshot.log 2>&1

# Manter apenas últimos 7 backups
find /mnt/backup -name "snapshot_*.tar.gz" -mtime +7 -delete
EOF

sudo chmod +x /etc/cron.daily/snapshot_backup

echo "Configuração de LVM snapshots concluída!"
```

#### **10.4.2 Recuperação a partir de Snapshot**
```bash
#!/bin/bash
# restore_from_snapshot.sh

# Configurações
BACKUP_FILE="$1"
RESTORE_DIR="/mnt/restore"
TARGET_DEVICE="/dev/vg_production/lv_production"

if [ -z "$BACKUP_FILE" ] || [ ! -f "$BACKUP_FILE" ]; then
    echo "Uso: $0 <arquivo_de_backup>"
    echo "Backups disponíveis:"
    find /mnt/backup -name "snapshot_*.tar.gz" | sort -r
    exit 1
fi

echo "=== RESTAURAÇÃO A PARTIR DE SNAPSHOT ==="
echo "Backup selecionado: $(basename $BACKUP_FILE)"
echo "Dispositivo alvo: $TARGET_DEVICE"
echo ""
echo "ATENÇÃO: Esta operação irá sobrescrever o sistema atual!"
echo ""

read -p "Confirmar restauração? (Digite 'RESTAURAR'): " confirm

if [ "$confirm" != "RESTAURAR" ]; then
    echo "Restauração cancelada"
    exit 0
fi

# 1. Entrar em modo de recuperação (single user)
echo "Entrando em modo de recuperação..."
sudo systemctl isolate rescue.target

# 2. Desmontar sistema de arquivos
echo "Desmontando sistema de arquivos..."
umount / 2>/dev/null || true
umount /home 2>/dev/null || true
umount /opt 2>/dev/null || true

# 3. Montar dispositivo alvo
echo "Montando dispositivo alvo..."
mount $TARGET_DEVICE $RESTORE_DIR

# 4. Extrair backup
echo "Extraindo backup..."
tar -xzf "$BACKUP_FILE" -C $RESTORE_DIR

# 5. Reinstalar bootloader
echo "Reinstalando bootloader..."
chroot $RESTORE_DIR grub-install /dev/sda
chroot $RESTORE_DIR update-grub

# 6. Desmontar
echo "Desmontando..."
umount $RESTORE_DIR

echo "Restauração concluída!"
echo "Reiniciando sistema em 10 segundos..."
sleep 10
reboot
```

---

## **11. MANUTENÇÃO E MONITORAMENTO**

### **11.1 Sistema de Monitoramento com Zabbix**

#### **11.1.1 Instalar e Configurar Zabbix Agent**
```bash
#!/bin/bash
# install_zabbix_agent.sh

# Adicionar repositório do Zabbix
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update

# Instalar Zabbix Agent 2
sudo apt install -y zabbix-agent2 zabbix-agent2-plugin-*

# Configurar Zabbix Agent
sudo tee /etc/zabbix/zabbix_agent2.conf << 'EOF'
# Configurações básicas
Server=192.168.1.50           # IP do servidor Zabbix
ServerActive=192.168.1.50
Hostname=production-server
HostMetadata=production,postgresql,redis,nginx

# Segurança
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=production_pointer_psk
TLSPSKFile=/etc/zabbix/zabbix_agent2.psk

# Logging
LogType=file
LogFile=/var/log/zabbix/zabbix_agent2.log
LogFileSize=100
DebugLevel=3

# Timeouts
Timeout=30

# Plugins habilitados
Plugins.Postgres.AllowedDB=production_pointer
Plugins.Redis.Uri=tcp://localhost:6379
Plugins.Redis.Password=R3d!sS3nh@2024
Plugins.Nginx.Urls=http://localhost/nginx_status

# User parameters (customizados)
Include=/etc/zabbix/zabbix_agent2.d/*.conf
EOF

# Gerar PSK (Pre-Shared Key)
sudo sh -c "openssl rand -hex 32 > /etc/zabbix/zabbix_agent2.psk"
sudo chown zabbix:zabbix /etc/zabbix/zabbix_agent2.psk
sudo chmod 600 /etc/zabbix/zabbix_agent2.psk

# Criar parâmetros customizados
sudo tee /etc/zabbix/zabbix_agent2.d/production_pointer.conf << 'EOF'
# Monitoramento específico do Production Pointer

# Verificar se aplicação está rodando
UserParameter=production.pointer.process,pgrep -f "gunicorn.*productionpointer" | wc -l

# Contar usuários ativos
UserParameter=production.pointer.active_users,psql -U production_user -d production_pointer -t -c "SELECT COUNT(*) FROM users WHERE last_login > NOW() - INTERVAL '15 minutes';"

# Contar OPs em produção
UserParameter=production.pointer.active_ops,psql -U production_user -d production_pointer -t -c "SELECT COUNT(*) FROM production_orders WHERE status = 'in_progress';"

# Contar apontamentos hoje
UserParameter=production.pointer.today_pointings,psql -U production_user -d production_pointer -t -c "SELECT COUNT(*) FROM production_pointing WHERE DATE(start_time) = CURRENT_DATE;"

# Verificar espaço em disco da aplicação
UserParameter=production.pointer.disk_usage,df -h /opt/productionpointer | awk 'NR==2 {print $5}' | tr -d '%'

# Verificar tamanho do banco de dados
UserParameter=production.pointer.db_size,psql -U production_user -d production_pointer -t -c "SELECT pg_database_size('production_pointer') / 1024 / 1024;"

# Verificar fila do Celery
UserParameter=production.pointer.celery_queue,redis-cli -a R3d!sS3nh@2024 LLEN celery 2>/dev/null || echo 0

# Verificar sincronização com ERP
UserParameter=production.pointer.erp_sync_status,psql -U production_user -d production_pointer -t -c "SELECT COUNT(*) FROM production_pointing WHERE sync_status = 'pending' AND created_at > NOW() - INTERVAL '1 hour';"

# Verificar logs de erro recentes
UserParameter=production.pointer.error_logs,tail -100 /opt/productionpointer/data/logs/app_stderr.log | grep -c "ERROR"

# Verificar performance da aplicação
UserParameter=production.pointer.response_time,curl -s -o /dev/null -w "%{time_total}" http://localhost:8000/health

# Verificar backups recentes
UserParameter=production.pointer.last_backup,find /opt/productionpointer/data/backups/daily -type d -name "production_pointer_*" | sort -r | head -1 | xargs basename 2>/dev/null || echo "none"
EOF

# Habilitar e iniciar serviço
sudo systemctl enable zabbix-agent2
sudo systemctl start zabbix-agent2
sudo systemctl status zabbix-agent2

# Configurar firewall para Zabbix
sudo ufw allow from 192.168.1.50 to any port 10050

echo "Zabbix Agent configurado!"
echo "PSK: $(cat /etc/zabbix/zabbix_agent2.psk)"
echo "Adicione este host ao servidor Zabbix com PSK identity: production_pointer_psk"
```

#### **11.1.2 Templates Zabbix para Production Pointer**
```xml
<!-- production_pointer_template.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<zabbix_export>
    <version>6.4</version>
    <date>2024-01-15T10:00:00Z</date>
    <groups>
        <group>
            <name>Templates/Applications</name>
        </group>
    </groups>
    <templates>
        <template>
            <template>Production Pointer Pro</template>
            <name>Production Pointer Pro</name>
            <description>Monitoramento completo do sistema Production Pointer Pro</description>
            <groups>
                <group>
                    <name>Templates/Applications</name>
                </group>
            </groups>
            
            <applications>
                <application>
                    <name>PostgreSQL</name>
                </application>
                <application>
                    <name>Redis</name>
                </application>
                <application>
                    <name>Nginx</name>
                </application>
                <application>
                    <name>Application</name>
                </application>
                <application>
                    <name>System</name>
                </application>
            </applications>
            
            <items>
                <!-- Processos da aplicação -->
                <item>
                    <name>Production Pointer: Processos ativos</name>
                    <type>ZABBIX_ACTIVE</type>
                    <key>production.pointer.process</key>
                    <delay>1m</delay>
                    <history>7d</history>
                    <trends>365d</trends>
                    <applications>
                        <application>
                            <name>Application</name>
                        </application>
                    </applications>
                    <triggers>
                        <trigger>
                            <expression>{Production Pointer Pro:production.pointer.process.last()}=0</expression>
                            <name>Production Pointer não está rodando</name>
                            <priority>HIGH</priority>
                        </trigger>
                    </triggers>
                </item>
                
                <!-- Usuários ativos -->
                <item>
                    <name>Production Pointer: Usuários ativos (15min)</name>
                    <type>ZABBIX_ACTIVE</type>
                    <key>production.pointer.active_users</key>
                    <delay>5m</delay>
                    <applications>
                        <application>
                            <name>Application</name>
                        </application>
                    </applications>
                </item>
                
                <!-- OPs em produção -->
                <item>
                    <name>Production Pointer: OPs em produção</name>
                    <type>ZABBIX_ACTIVE</type>
                    <key>production.pointer.active_ops</key>
                    <delay>5m</delay>
                    <applications>
                        <application>
                            <name>Application</name>
                        </application>
                    </applications>
                    <triggers>
                        <trigger>
                            <expression>{Production Pointer Pro:production.pointer.active_ops.last()}=0</expression>
                            <name>Nenhuma OP em produção</name>
                            <priority>WARNING</priority>
                        </trigger>
                    </triggers>
                </item>
                
                <!-- Apontamentos hoje -->
                <item>
                    <name>Production Pointer: Apontamentos hoje</name>
                    <type>ZABBIX_ACTIVE</type>
                    <key>production.pointer.today_pointings</key>
                    <delay>15m</delay>
                    <applications>
                        <application>
                            <name>Application</name>
                        </application>
                    </applications>
                </item>
                
                <!-- Tamanho do banco -->
                <item>
                    <name>Production Pointer: Tamanho do banco (MB)</name>
                    <type>ZABBIX_ACTIVE</type>
                    <key>production.pointer.db_size</key>
                    <units>MB</units>
                    <delay>1h</delay>
                    <applications>
                        <application>
                            <name>PostgreSQL</name>
                        </application>
                    </applications>
                    <triggers>
                        <trigger>
                            <expression>{Production Pointer Pro:production.pointer.db_size.last()}>10000</expression>
                            <name>Banco de dados maior que 10GB</name>
                            <priority>WARNING</priority>
                        </trigger>
                    </triggers>
                </item>
                
                <!-- Fila do Celery -->
                <item>
                    <name>Production Pointer: Tarefas na fila Celery</name>
                    <type>ZABBIX_ACTIVE</type>
                    <key>production.pointer.celery_queue</key>
                    <delay>5m</delay>
                    <applications>
                        <application>
                            <name>Application</name>
                        </application>
                    </applications>
                    <triggers>
                        <trigger>
                            <expression>{Production Pointer Pro:production.pointer.celery_queue.last()}>100</expression>
                            <name>Fila Celery com mais de 100 tarefas</name>
                            <priority>WARNING</priority>
                        </trigger>
                    </triggers>
                </item>
                
                <!-- Sincronização ERP -->
                <item>
                    <name>Production Pointer: Apontamentos pendentes para ERP</name>
                    <type>ZABBIX_ACTIVE</type>
                    <key>production.pointer.erp_sync_status</key>
                    <delay>5m</delay>
                    <applications>
                        <application>
                            <name>Application</name>
                        </application>
                    </applications>
                    <triggers>
                        <trigger>
                            <expression>{Production Pointer Pro:production.pointer.erp_sync_status.last()}>50</expression>
                            <name>Mais de 50 apontamentos pendentes para ERP</name>
                            <priority>WARNING</priority>
                        </trigger>
                    </triggers>
                </item>
                
                <!-- Logs de erro -->
                <item>
                    <name>Production Pointer: Erros nos logs (últimas 100 linhas)</name>
                    <type>ZABBIX_ACTIVE</type>
                    <key>production.pointer.error_logs</key>
                    <delay>5m</delay>
                    <applications>
                        <application>
                            <name>Application</name>
                        </application>
                    </applications>
                    <triggers>
                        <trigger>
                            <expression>{Production Pointer Pro:production.pointer.error_logs.last()}>10</expression>
                            <name>Mais de 10 erros nos logs</name>
                            <priority>WARNING</priority>
                        </trigger>
                    </triggers>
                </item>
                
                <!-- Tempo de resposta -->
                <item>
                    <name>Production Pointer: Tempo de resposta (segundos)</name>
                    <type>ZABBIX_ACTIVE</type>
                    <key>production.pointer.response_time</key>
                    <units>s</units>
                    <delay>1m</delay>
                    <applications>
                        <application>
                            <name>Application</name>
                        </application>
                    </applications>
                    <triggers>
                        <trigger>
                            <expression>{Production Pointer Pro:production.pointer.response_time.last()}>5</expression>
                            <name>Tempo de resposta maior que 5 segundos</name>
                            <priority>WARNING</priority>
                        </trigger>
                        <trigger>
                            <expression>{Production Pointer Pro:production.pointer.response_time.last()}>10</expression>
                            <name>Tempo de resposta maior que 10 segundos</name>
                            <priority>HIGH</priority>
                        </trigger>
                    </triggers>
                </item>
            </items>
            
            <graphs>
                <graph>
                    <name>Production Pointer: Visão Geral</name>
                    <width>900</width>
                    <height>200</height>
                    <graph_items>
                        <graph_item>
                            <item>
                                <host>Production Pointer Pro</host>
                                <key>production.pointer.process</key>
                            </item>
                            <color>00FF00</color>
                        </graph_item>
                        <graph_item>
                            <item>
                                <host>Production Pointer Pro</host>
                                <key>production.pointer.active_ops</key>
                            </item>
                            <color>FF0000</color>
                        </graph_item>
                        <graph_item>
                            <item>
                                <host>Production Pointer Pro</host>
                                <key>production.pointer.active_users</key>
                            </item>
                            <color>0000FF</color>
                        </graph_item>
                    </graph_items>
                </graph>
                
                <graph>
                    <name>Production Pointer: Performance</name>
                    <width>900</width>
                    <height>200</height>
                    <graph_items>
                        <graph_item>
                            <item>
                                <host>Production Pointer Pro</host>
                                <key>production.pointer.response_time</key>
                            </item>
                            <color>FFA500</color>
                        </graph_item>
                        <graph_item>
                            <item>
                                <host>Production Pointer Pro</host>
                                <key>production.pointer.celery_queue</key>
                            </item>
                            <color>800080</color>
                        </graph_item>
                        <graph_item>
                            <item>
                                <host>Production Pointer Pro</host>
                                <key>production.pointer.erp_sync_status</key>
                            </item>
                            <color>008080</color>
                        </graph_item>
                    </graph_items>
                </graph>
            </graphs>
            
            <dashboards>
                <dashboard>
                    <name>Production Pointer Dashboard</name>
                    <pages>
                        <page>
                            <name>Visão Geral</name>
                            <widgets>
                                <widget>
                                    <type>problemhosts</type>
                                    <x>0</x>
                                    <y>0</y>
                                    <width>12</width>
                                    <height>5</height>
                                </widget>
                                <widget>
                                    <type>graph</type>
                                    <x>0</x>
                                    <y>5</y>
                                    <width>12</width>
                                    <height>6</height>
                                    <fields>
                                        <field>
                                            <type>INTEGER</type>
                                            <name>resourceid</name>
                                            <value>1001</value> <!-- ID do gráfico Visão Geral -->
                                        </field>
                                    </fields>
                                </widget>
                            </widgets>
                        </page>
                    </pages>
                </dashboard>
            </dashboards>
        </template>
    </templates>
</zabbix_export>
```

### **11.2 Scripts de Manutenção Automática**

#### **11.2.1 Limpeza e Otimização do Banco de Dados**
```bash
#!/bin/bash
# database_maintenance.sh

# Configurações
LOG_FILE="/opt/productionpointer/data/logs/db_maintenance.log"
RETENTION_DAYS=180  # Manter 6 meses de dados detalhados
ARCHIVE_DAYS=30     # Arquivar após 30 dias

# Função para logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Função para executar comando PostgreSQL
exec_sql() {
    local sql="$1"
    sudo -u postgres psql -d production_pointer -c "$sql"
}

# Iniciar manutenção
log "=== INICIANDO MANUTENÇÃO DO BANCO DE DADOS ==="

# 1. Estatísticas e VACUUM
log "Executando ANALYZE e VACUUM..."
exec_sql "VACUUM ANALYZE;"

# 2. Reindexar tabelas grandes
log "Reindexando tabelas grandes..."
exec_sql "REINDEX TABLE production_pointing;"
exec_sql "REINDEX TABLE machine_stops;"
exec_sql "REINDEX TABLE audit_logs;"

# 3. Arquivar dados antigos de produção_pointing
log "Arquivando dados antigos de produção..."
ARCHIVE_DATE=$(date -d "$ARCHIVE_DAYS days ago" +%Y-%m-%d)

exec_sql "
-- Criar tabela de arquivamento se não existir
CREATE TABLE IF NOT EXISTS production_pointing_archive 
    (LIKE production_pointing INCLUDING ALL);

-- Inserir dados antigos na tabela de arquivamento
INSERT INTO production_pointing_archive 
SELECT * FROM production_pointing 
WHERE start_time < '$ARCHIVE_DATE'::date;

-- Remover dados arquivados
DELETE FROM production_pointing 
WHERE start_time < '$ARCHIVE_DATE'::date;

-- Compactar tabela de arquivamento
VACUUM FULL production_pointing_archive;
"

ARCHIVED_COUNT=$(exec_sql "SELECT COUNT(*) FROM production_pointing_archive;" | tail -3 | head -1 | tr -d ' ')
log "Dados arquivados: $ARCHIVED_COUNT registros"

# 4. Remover dados muito antigos
log "Removendo dados muito antigos..."
DELETE_DATE=$(date -d "$RETENTION_DAYS days ago" +%Y-%m-%d)

DELETED_COUNT=$(exec_sql "
    DELETE FROM production_pointing_archive 
    WHERE start_time < '$DELETE_DATE'::date 
    RETURNING COUNT(*);
" | tail -3 | head -1 | tr -d ' ')

log "Dados removidos: $DELETED_COUNT registros"

# 5. Otimizar tabelas de logs
log "Otimizando tabelas de logs..."
exec_sql "
-- Criar índice para logs antigos se não existir
CREATE INDEX IF NOT EXISTS idx_audit_logs_old 
    ON audit_logs (created_at) 
    WHERE created_at < NOW() - INTERVAL '30 days';

-- Compactar logs antigos
VACUUM FULL audit_logs;
"

# 6. Estatísticas de espaço
log "Coletando estatísticas de espaço..."
exec_sql "
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) as total_size,
    pg_size_pretty(pg_relation_size(schemaname || '.' || tablename)) as table_size,
    pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename) - 
                   pg_relation_size(schemaname || '.' || tablename)) as index_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname || '.' || tablename) DESC
LIMIT 10;
" | tee -a "$LOG_FILE"

# 7. Verificar bloat (espaço desperdiçado)
log "Verificando bloat nas tabelas..."
exec_sql "
SELECT
    schemaname,
    tablename,
    ROUND((CASE WHEN otta=0 THEN 0.0 ELSE sml.relpages::FLOAT/otta END)::NUMERIC,1) AS tbloat,
    CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::BIGINT END AS wastedbytes,
    iname,
    ROUND((CASE WHEN iotta=0 OR ipages=0 THEN 0.0 ELSE ipages::FLOAT/iotta END)::NUMERIC,1) AS ibloat,
    CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes
FROM (
    SELECT
        schemaname, tablename, cc.reltuples, cc.relpages, bs,
        CEIL((cc.reltuples*((datahdr+ma-
            (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::FLOAT)) AS otta,
        COALESCE(c2.relname,'?') AS iname, COALESCE(c2.reltuples,0) AS ituples, 
        COALESCE(c2.relpages,0) AS ipages,
        COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::FLOAT)),0) AS iotta
    FROM (
        SELECT
            ma,bs,schemaname,tablename,
            (datawidth+(hdr+ma-(CASE WHEN hdr%ma=0 THEN ma ELSE hdr%ma END)))::NUMERIC AS datahdr,
            (maxfracsum*(nullhdr+ma-(CASE WHEN nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2
        FROM (
            SELECT
                schemaname, tablename, hdr, ma, bs,
                SUM((1-null_frac)*avg_width) AS datawidth,
                MAX(null_frac) AS maxfracsum,
                hdr+(
                    SELECT 1+COUNT(*)/8
                    FROM pg_stats s2
                    WHERE null_frac<>0 AND s2.schemaname = s.schemaname 
                    AND s2.tablename = s.tablename
                ) AS nullhdr
            FROM pg_stats s, (
                SELECT
                    (SELECT current_setting('block_size')::NUMERIC) AS bs,
                    CASE WHEN SUBSTRING(v,12,3) IN ('8.0','8.1','8.2') THEN 27 ELSE 23 END AS hdr,
                    CASE WHEN v ~ 'mingw32' THEN 8 ELSE 4 END AS ma
                FROM (SELECT version() AS v) AS foo
            ) AS constants
            GROUP BY 1,2,3,4,5
        ) AS foo
    ) AS rs
    JOIN pg_class cc ON cc.relname = rs.tablename
    JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname = rs.schemaname
    LEFT JOIN pg_index i ON indrelid = cc.oid
    LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid
) AS sml
WHERE sml.relpages > 0 
    AND (sml.relpages - otta) * bs > 1000000  -- Mais de 1MB desperdiçado
ORDER BY wastedbytes DESC
LIMIT 10;
" | tee -a "$LOG_FILE"

# 8. Backup das estatísticas
log "Fazendo backup das estatísticas..."
STATS_BACKUP="/opt/productionpointer/data/backups/db_stats_$(date +%Y%m%d).sql"
exec_sql "SELECT pg_stat_reset();"  # Resetar estatísticas após backup

# 9. Verificar integridade
log "Verificando integridade do banco..."
exec_sql "
SELECT 
    schemaname,
    tablename,
    (SELECT COUNT(*) FROM pg_indexes WHERE tablename = pt.tablename) as indexes_count,
    (SELECT COUNT(*) FROM pg_constraints WHERE conrelid = pc.oid) as constraints_count
FROM pg_tables pt
JOIN pg_class pc ON pt.tablename = pc.relname
WHERE pt.schemaname = 'public'
ORDER BY pt.tablename;
" | tee -a "$LOG_FILE"

log "=== MANUTENÇÃO DO BANCO CONCLUÍDA ==="

# 10. Notificar conclusão
echo "Manutenção do banco concluída em $(date)" | \
    mail -s "Manutenção DB OK" admin@empresa.com.br
```

#### **11.2.2 Manutenção do Sistema de Arquivos**
```bash
#!/bin/bash
# filesystem_maintenance.sh

# Configurações
LOG_FILE="/opt/productionpointer/data/logs/fs_maintenance.log"
UPLOADS_DIR="/opt/productionpointer/data/uploads"
LOGS_DIR="/opt/productionpointer/data/logs"
TEMP_DIR="/opt/productionpointer/data/temp"

# Função para logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Função para verificar espaço
check_space() {
    local dir="$1"
    local threshold="$2"  # Porcentagem
    
    local usage=$(df "$dir" | awk 'NR==2 {print $5}' | tr -d '%')
    
    if [ "$usage" -ge "$threshold" ]; then
        log "ALERTA: Espaço em $dir está em ${usage}% (limite: ${threshold}%)"
        return 1
    fi
    
    return 0
}

# Iniciar manutenção
log "=== INICIANDO MANUTENÇÃO DO SISTEMA DE ARQUIVOS ==="

# 1. Verificar espaço em disco
log "Verificando espaço em disco..."
check_space "/opt" 80
check_space "/var" 80
check_space "/tmp" 90

# 2. Limpar arquivos temporários
log "Limpando arquivos temporários..."
find "$TEMP_DIR" -type f -mtime +1 -delete
find "$TEMP_DIR" -type d -empty -delete
find "/tmp" -name "production_pointer_*" -mtime +1 -delete

# 3. Rotacionar logs grandes
log "Rotacionando logs grandes..."
for logfile in "$LOGS_DIR"/*.log; do
    if [ -f "$logfile" ] && [ $(stat -c%s "$logfile") -gt 104857600 ]; then  # > 100MB
        log "Rotacionando log grande: $(basename $logfile)"
        mv "$logfile" "${logfile}.old"
        touch "$logfile"
    fi
done

# 4. Limpar logs antigos
log "Limpando logs antigos..."
find "$LOGS_DIR" -name "*.log.old" -mtime +30 -delete
find "$LOGS_DIR" -name "*.log.*" -mtime +90 -delete
find "/var/log" -name "production-pointer*" -mtime +30 -delete

# 5. Limpar uploads antigos
log "Limpando uploads antigos..."
# Fotos de paradas com mais de 90 dias
find "$UPLOADS_DIR/photos" -type f -mtime +90 -delete
find "$UPLOADS_DIR/photos" -type d -empty -delete

# QR Codes antigos (exceto os ativos)
find "$UPLOADS_DIR/qrcodes" -type f -mtime +180 -delete

# Relatórios exportados com mais de 180 dias
find "$UPLOADS_DIR/reports" -type f -mtime +180 -delete

# 6. Verificar permissões
log "Verificando permissões..."
find "/opt/productionpointer/app" -type f ! -perm 640 -exec chmod 640 {} \;
find "/opt/productionpointer/app" -type d ! -perm 750 -exec chmod 750 {} \;
find "/opt/productionpointer/data" -type f ! -perm 640 -exec chmod 640 {} \;
find "/opt/productionpointer/data" -type d ! -perm 750 -exec chmod 750 {} \;

# 7. Verificar propriedade
log "Verificando propriedade dos arquivos..."
find "/opt/productionpointer" ! -user production -exec chown production:production {} \;

# 8. Compactar logs antigos
log "Compactando logs antigos..."
find "$LOGS_DIR" -name "*.log" -mtime +7 -exec gzip {} \;

# 9. Verificar inodes
log "Verificando uso de inodes..."
INODE_USAGE=$(df -i /opt | awk 'NR==2 {print $5}' | tr -d '%')
log "Uso de inodes em /opt: ${INODE_USAGE}%"

if [ "$INODE_USAGE" -gt 80 ]; then
    log "ALERTA: Uso de inodes está alto!"
    
    # Encontrar diretórios com muitos arquivos
    find /opt/productionpointer -type d | while read dir; do
        count=$(find "$dir" -maxdepth 1 -type f | wc -l)
        if [ "$count" -gt 1000 ]; then
            log "Diretório com muitos arquivos: $dir ($count arquivos)"
        fi
    done
fi

# 10. Estatísticas finais
log "Estatísticas finais:"
df -h /opt /var /tmp | tee -a "$LOG_FILE"
du -sh /opt/productionpointer/* | tee -a "$LOG_FILE"

log "=== MANUTENÇÃO DO SISTEMA DE ARQUIVOS CONCLUÍDA ==="
```

#### **11.2.3 Monitoramento de Performance**
```bash
#!/bin/bash
# performance_monitoring.sh

# Configurações
LOG_FILE="/opt/productionpointer/data/logs/performance.log"
REPORT_FILE="/opt/productionpointer/data/logs/performance_report_$(date +%Y%m%d).txt"
ALERT_THRESHOLDS=(
    "CPU:90"
    "MEMORY:85" 
    "DISK:80"
    "LOAD:$(nproc)"
    "CONNECTIONS:150"
)

# Função para logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Função para enviar alerta
send_alert() {
    local metric="$1"
    local value="$2"
    local threshold="$3"
    
    local subject="ALERTA: $metric acima do limite"
    local message="Métrica: $metric\nValor: $value\nLimite: $threshold\nHorário: $(date)"
    
    echo -e "$message" | mail -s "$subject" admin@empresa.com.br
    log "ALERTA ENVIADO: $metric = $value (limite: $threshold)"
}

# Função para verificar threshold
check_threshold() {
    local metric="$1"
    local value="$2"
    
    for threshold in "${ALERT_THRESHOLDS[@]}"; do
        local t_metric="${threshold%%:*}"
        local t_value="${threshold##*:}"
        
        if [ "$metric" = "$t_metric" ]; then
            if (( $(echo "$value > $t_value" | bc -l) )); then
                send_alert "$metric" "$value" "$t_value"
                return 1
            fi
        fi
    done
    
    return 0
}

# Iniciar monitoramento
log "=== INICIANDO MONITORAMENTO DE PERFORMANCE ==="

# 1. CPU Usage
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}' | cut -d'.' -f1)
log "CPU Usage: ${CPU_USAGE}%"
check_threshold "CPU" "$CPU_USAGE"

# 2. Memory Usage
MEMORY_USAGE=$(free | awk '/Mem:/ {print ($3/$2)*100}' | cut -d'.' -f1)
log "Memory Usage: ${MEMORY_USAGE}%"
check_threshold "MEMORY" "$MEMORY_USAGE"

# 3. Load Average
LOAD_AVG=$(uptime | awk -F'load average:' '{print $2}' | cut -d',' -f1 | tr -d ' ')
CPU_COUNT=$(nproc)
log "Load Average: $LOAD_AVG (CPUs: $CPU_COUNT)"
check_threshold "LOAD" "$LOAD_AVG"

# 4. Disk Usage
DISK_USAGE=$(df /opt | awk 'NR==2 {print $5}' | tr -d '%')
log "Disk Usage (/opt): ${DISK_USAGE}%"
check_threshold "DISK" "$DISK_USAGE"

# 5. PostgreSQL Connections
PG_CONNECTIONS=$(sudo -u postgres psql -t -c "SELECT COUNT(*) FROM pg_stat_activity WHERE datname = 'production_pointer';" | tr -d ' ')
log "PostgreSQL Connections: $PG_CONNECTIONS"
check_threshold "CONNECTIONS" "$PG_CONNECTIONS"

# 6. Redis Memory
REDIS_MEMORY=$(redis-cli -a "$REDIS_PASSWORD" info memory | grep "used_memory_human" | cut -d':' -f2 | tr -d '\r')
log "Redis Memory Usage: $REDIS_MEMORY"

# 7. Nginx Connections
NGINX_CONNECTIONS=$(netstat -an | grep :443 | grep ESTABLISHED | wc -l)
log "Nginx Active Connections: $NGINX_CONNECTIONS"

# 8. Application Response Time
RESPONSE_TIME=$(curl -s -o /dev/null -w "%{time_total}" http://localhost:8000/health)
log "Application Response Time: ${RESPONSE_TIME}s"

if (( $(echo "$RESPONSE_TIME > 5" | bc -l) )); then
    send_alert "RESPONSE_TIME" "$RESPONSE_TIME" "5"
fi

# 9. Celery Queue
CELERY_QUEUE=$(redis-cli -a "$REDIS_PASSWORD" LLEN celery 2>/dev/null || echo "0")
log "Celery Queue Length: $CELERY_QUEUE"

if [ "$CELERY_QUEUE" -gt 100 ]; then
    send_alert "CELERY_QUEUE" "$CELERY_QUEUE" "100"
fi

# 10. Database Size
DB_SIZE=$(sudo -u postgres psql -t -c "SELECT pg_database_size('production_pointer') / 1024 / 1024;" | tr -d ' ')
log "Database Size: ${DB_SIZE}MB"

# 11. Gerar relatório
cat > "$REPORT_FILE" << EOF
=== RELATÓRIO DE PERFORMANCE ===
Data: $(date)

1. SISTEMA
CPU Usage: ${CPU_USAGE}%
Memory Usage: ${MEMORY_USAGE}%
Load Average: $LOAD_AVG
Disk Usage (/opt): ${DISK_USAGE}%

2. SERVIÇOS
PostgreSQL Connections: $PG_CONNECTIONS
Redis Memory: $REDIS_MEMORY
Nginx Connections: $NGINX_CONNECTIONS

3. APLICAÇÃO
Response Time: ${RESPONSE_TIME}s
Celery Queue: $CELERY_QUEUE
Database Size: ${DB_SIZE}MB

4. RECOMENDAÇÕES
EOF

# Adicionar recomendações baseadas nos dados
if [ "$CPU_USAGE" -gt 70 ]; then
    echo "- Considerar upgrade de CPU ou otimização de queries" >> "$REPORT_FILE"
fi

if [ "$MEMORY_USAGE" -gt 80 ]; then
    echo "- Considerar aumento de memória RAM" >> "$REPORT_FILE"
fi

if [ "$DISK_USAGE" -gt 70 ]; then
    echo "- Limpar dados antigos ou aumentar espaço em disco" >> "$REPORT_FILE"
fi

if [ "$CELERY_QUEUE" -gt 50 ]; then
    echo "- Aumentar workers do Celery" >> "$REPORT_FILE"
fi

if (( $(echo "$RESPONSE_TIME > 3" | bc -l) )); then
    echo "- Otimizar performance da aplicação" >> "$REPORT_FILE"
fi

log "Relatório gerado: $REPORT_FILE"

# 12. Enviar relatório diário
if [ $(date +%H) -eq 8 ]; then  # Somente às 8h
    mail -s "Relatório de Performance - $(date +%Y-%m-%d)" admin@empresa.com.br < "$REPORT_FILE"
    log "Relatório enviado por email"
fi

log "=== MONITORAMENTO CONCLUÍDO ==="
```

### **11.3 Dashboard de Monitoramento Personalizado**

#### **11.3.1 API de Status do Sistema**
```python
# app/api/status.py
from flask import Blueprint, jsonify
from datetime import datetime, timedelta
import psutil
import redis
import psycopg2
from .. import db

status_bp = Blueprint('status', __name__)

@status_bp.route('/api/status/health', methods=['GET'])
def health_check():
    """Endpoint básico de health check"""
    return jsonify({
        'status': 'healthy',
        'timestamp': datetime.utcnow().isoformat(),
        'version': '1.0.0'
    })

@status_bp.route('/api/status/detailed', methods=['GET'])
def detailed_status():
    """Status detalhado do sistema"""
    
    status = {
        'timestamp': datetime.utcnow().isoformat(),
        'components': {},
        'metrics': {},
        'alerts': []
    }
    
    # 1. Sistema Operacional
    try:
        # CPU
        cpu_percent = psutil.cpu_percent(interval=1)
        cpu_count = psutil.cpu_count()
        
        # Memória
        memory = psutil.virtual_memory()
        memory_percent = memory.percent
        memory_used_gb = memory.used / (1024**3)
        memory_total_gb = memory.total / (1024**3)
        
        # Disco
        disk = psutil.disk_usage('/opt')
        disk_percent = disk.percent
        disk_used_gb = disk.used / (1024**3)
        disk_total_gb = disk.total / (1024**3)
        
        # Load Average
        load_avg = psutil.getloadavg()
        
        status['components']['system'] = {
            'cpu': {
                'percent': cpu_percent,
                'cores': cpu_count,
                'load_average': load_avg
            },
            'memory': {
                'percent': memory_percent,
                'used_gb': round(memory_used_gb, 2),
                'total_gb': round(memory_total_gb, 2)
            },
            'disk': {
                'percent': disk_percent,
                'used_gb': round(disk_used_gb, 2),
                'total_gb': round(disk_total_gb, 2)
            }
        }
        
        # Alertas do sistema
        if cpu_percent > 90:
            status['alerts'].append({
                'component': 'system.cpu',
                'level': 'warning',
                'message': f'CPU usage high: {cpu_percent}%'
            })
        
        if memory_percent > 85:
            status['alerts'].append({
                'component': 'system.memory',
                'level': 'warning',
                'message': f'Memory usage high: {memory_percent}%'
            })
            
    except Exception as e:
        status['components']['system'] = {'error': str(e)}
    
    # 2. PostgreSQL
    try:
        conn = db.engine.connect()
        
        # Conexões ativas
        result = conn.execute("""
            SELECT COUNT(*) as active_connections,
                   COUNT(CASE WHEN state = 'active' THEN 1 END) as active_queries,
                   COUNT(CASE WHEN state = 'idle' THEN 1 END) as idle_connections
            FROM pg_stat_activity
            WHERE datname = 'production_pointer'
        """).fetchone()
        
        # Tamanho do banco
        size_result = conn.execute("""
            SELECT pg_database_size('production_pointer') / 1024 / 1024 as size_mb
        """).fetchone()
        
        # Locks ativos
        locks_result = conn.execute("""
            SELECT COUNT(*) as locks_count
            FROM pg_locks
            WHERE granted = false
        """).fetchone()
        
        conn.close()
        
        status['components']['postgresql'] = {
            'connections': {
                'active': result[0],
                'queries': result[1],
                'idle': result[2]
            },
            'database_size_mb': size_result[0],
            'waiting_locks': locks_result[0],
            'status': 'healthy'
        }
        
        if result[0] > 150:  # Mais de 150 conexões
            status['alerts'].append({
                'component': 'postgresql.connections',
                'level': 'warning',
                'message': f'High connection count: {result[0]}'
            })
            
    except Exception as e:
        status['components']['postgresql'] = {'error': str(e), 'status': 'unhealthy'}
    
    # 3. Redis
    try:
        r = redis.Redis(
            host='localhost',
            port=6379,
            password='R3d!sS3nh@2024',
            decode_responses=True
        )
        
        redis_info = r.info()
        
        status['components']['redis'] = {
            'used_memory_mb': int(redis_info['used_memory']) / (1024**2),
            'connected_clients': redis_info['connected_clients'],
            'keyspace_hits': redis_info['keyspace_hits'],
            'keyspace_misses': redis_info['keyspace_misses'],
            'hit_rate': redis_info['keyspace_hits'] / (redis_info['keyspace_hits'] + redis_info['keyspace_misses']) * 100 if (redis_info['keyspace_hits'] + redis_info['keyspace_misses']) > 0 else 0,
            'status': 'healthy'
        }
        
        if redis_info['used_memory'] > 1.5 * (1024**3):  # > 1.5GB
            status['alerts'].append({
                'component': 'redis.memory',
                'level': 'warning',
                'message': f'Redis memory high: {int(redis_info["used_memory"]) / (1024**3):.2f}GB'
            })
            
    except Exception as e:
        status['components']['redis'] = {'error': str(e), 'status': 'unhealthy'}
    
    # 4. Aplicação
    try:
        from ..models import ProductionOrder, ProductionPointing, User
        
        # Estatísticas da aplicação
        active_ops = ProductionOrder.query.filter_by(status='in_progress').count()
        active_users = User.query.filter(User.last_login > datetime.utcnow() - timedelta(minutes=15)).count()
        today_pointings = ProductionPointing.query.filter(
            db.func.date(ProductionPointing.start_time) == datetime.utcnow().date()
        ).count()
        
        # Pendências de sincronização
        pending_sync = ProductionPointing.query.filter_by(sync_status='pending').count()
        
        status['components']['application'] = {
            'active_operations': active_ops,
            'active_users': active_users,
            'today_pointings': today_pointings,
            'pending_sync': pending_sync,
            'status': 'healthy'
        }
        
        if pending_sync > 100:
            status['alerts'].append({
                'component': 'application.sync',
                'level': 'warning',
                'message': f'High pending sync count: {pending_sync}'
            })
            
    except Exception as e:
        status['components']['application'] = {'error': str(e), 'status': 'unhealthy'}
    
    # 5. Calcula status geral
    healthy_components = 0
    total_components = 0
    
    for component, data in status['components'].items():
        total_components += 1
        if data.get('status') == 'healthy' or 'error' not in data:
            healthy_components += 1
    
    overall_health = (healthy_components / total_components) * 100 if total_components > 0 else 0
    
    status['overall'] = {
        'health_percentage': round(overall_health, 2),
        'status': 'healthy' if overall_health > 80 else 'degraded' if overall_health > 50 else 'unhealthy',
        'healthy_components': healthy_components,
        'total_components': total_components
    }
    
    return jsonify(status)

@status_bp.route('/api/status/metrics', methods=['GET'])
def prometheus_metrics():
    """Endpoint para métricas no formato Prometheus"""
    metrics = []
    
    try:
        # Sistema
        cpu_percent = psutil.cpu_percent(interval=1)
        memory = psutil.virtual_memory()
        disk = psutil.disk_usage('/opt')
        
        metrics.append(f'# HELP system_cpu_percent CPU usage percentage')
        metrics.append(f'# TYPE system_cpu_percent gauge')
        metrics.append(f'system_cpu_percent {cpu_percent}')
        
        metrics.append(f'# HELP system_memory_percent Memory usage percentage')
        metrics.append(f'# TYPE system_memory_percent gauge')
        metrics.append(f'system_memory_percent {memory.percent}')
        
        metrics.append(f'# HELP system_disk_percent Disk usage percentage')
        metrics.append(f'# TYPE system_disk_percent gauge')
        metrics.append(f'system_disk_percent {disk.percent}')
        
        # PostgreSQL
        conn = db.engine.connect()
        pg_conns = conn.execute("SELECT COUNT(*) FROM pg_stat_activity WHERE datname = 'production_pointer'").scalar()
        conn.close()
        
        metrics.append(f'# HELP postgresql_connections Active PostgreSQL connections')
        metrics.append(f'# TYPE postgresql_connections gauge')
        metrics.append(f'postgresql_connections {pg_conns}')
        
        # Redis
        r = redis.Redis(
            host='localhost',
            port=6379,
            password='R3d!sS3nh@2024',
            decode_responses=True
        )
        redis_info = r.info()
        
        metrics.append(f'# HELP redis_used_memory_bytes Redis used memory in bytes')
        metrics.append(f'# TYPE redis_used_memory_bytes gauge')
        metrics.append(f'redis_used_memory_bytes {redis_info["used_memory"]}')
        
        # Aplicação
        from ..models import ProductionOrder, ProductionPointing
        
        active_ops = ProductionOrder.query.filter_by(status='in_progress').count()
        today_pointings = ProductionPointing.query.filter(
            db.func.date(ProductionPointing.start_time) == datetime.utcnow().date()
        ).count()
        
        metrics.append(f'# HELP application_active_ops Active production orders')
        metrics.append(f'# TYPE application_active_ops gauge')
        metrics.append(f'application_active_ops {active_ops}')
        
        metrics.append(f'# HELP application_today_pointings Pointings today')
        metrics.append(f'# TYPE application_today_pointings counter')
        metrics.append(f'application_today_pointings {today_pointings}')
        
    except Exception as e:
        metrics.append(f'# ERROR {str(e)}')
    
    return '\n'.join(metrics), 200, {'Content-Type': 'text/plain'}
```

#### **11.3.2 Dashboard HTML em Tempo Real**
```html
<!-- templates/admin/dashboard.html -->
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Production Pointer - Dashboard</title>
    
    <style>
        :root {
            --primary-color: #2c3e50;
            --secondary-color: #3498db;
            --success-color: #27ae60;
            --warning-color: #f39c12;
            --danger-color: #e74c3c;
            --light-color: #ecf0f1;
            --dark-color: #2c3e50;
        }
        
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f5f7fa;
            color: #333;
        }
        
        .dashboard {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            padding: 20px;
            max-width: 1400px;
            margin: 0 auto;
        }
        
        .card {
            background: white;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            padding: 20px;
            transition: transform 0.3s;
        }
        
        .card:hover {
            transform: translateY(-5px);
        }
        
        .card-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
            padding-bottom: 10px;
            border-bottom: 2px solid var(--light-color);
        }
        
        .card-title {
            font-size: 1.2rem;
            font-weight: 600;
            color: var(--primary-color);
        }
        
        .card-status {
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 0.8rem;
            font-weight: 600;
        }
        
        .status-healthy {
            background-color: rgba(39, 174, 96, 0.1);
            color: var(--success-color);
        }
        
        .status-warning {
            background-color: rgba(243, 156, 18, 0.1);
            color: var(--warning-color);
        }
        
        .status-danger {
            background-color: rgba(231, 76, 60, 0.1);
            color: var(--danger-color);
        }
        
        .metric {
            margin: 10px 0;
        }
        
        .metric-label {
            display: flex;
            justify-content: space-between;
            margin-bottom: 5px;
        }
        
        .metric-name {
            font-size: 0.9rem;
            color: #666;
        }
        
        .metric-value {
            font-weight: 600;
            color: var(--primary-color);
        }
        
        .progress-bar {
            height: 8px;
            background-color: var(--light-color);
            border-radius: 4px;
            overflow: hidden;
        }
        
        .progress-fill {
            height: 100%;
            border-radius: 4px;
            transition: width 0.5s ease;
        }
        
        .progress-cpu { background-color: var(--secondary-color); }
        .progress-memory { background-color: var(--success-color); }
        .progress-disk { background-color: var(--warning-color); }
        .progress-redis { background-color: var(--danger-color); }
        
        .alert-panel {
            grid-column: 1 / -1;
        }
        
        .alert {
            padding: 10px;
            margin: 5px 0;
            border-radius: 5px;
            display: flex;
            align-items: center;
            animation: fadeIn 0.5s;
        }
        
        .alert-warning {
            background-color: rgba(243, 156, 18, 0.1);
            border-left: 4px solid var(--warning-color);
        }
        
        .alert-danger {
            background-color: rgba(231, 76, 60, 0.1);
            border-left: 4px solid var(--danger-color);
        }
        
        .alert-icon {
            margin-right: 10px;
            font-size: 1.2rem;
        }
        
        .chart-container {
            height: 200px;
            margin-top: 15px;
        }
        
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
        }
        
        .stat-box {
            text-align: center;
            padding: 15px;
            border-radius: 8px;
            background-color: var(--light-color);
        }
        
        .stat-value {
            font-size: 2rem;
            font-weight: 700;
            color: var(--primary-color);
        }
        
        .stat-label {
            font-size: 0.8rem;
            color: #666;
            margin-top: 5px;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        
        .refresh-info {
            text-align: center;
            color: #999;
            font-size: 0.8rem;
            margin-top: 20px;
            grid-column: 1 / -1;
        }
    </style>
    
    <!-- Chart.js para gráficos -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <!-- Socket.io para atualização em tempo real -->
    <script src="https://cdn.socket.io/4.5.4/socket.io.min.js"></script>
</head>
<body>
    <div class="dashboard">
        <!-- Cabeçalho -->
        <div class="card alert-panel">
            <div class="card-header">
                <h2 class="card-title">Production Pointer - Dashboard</h2>
                <div class="card-status" id="overall-status">Carregando...</div>
            </div>
            <div id="alerts-container">
                <!-- Alertas serão inseridos aqui -->
            </div>
        </div>
        
        <!-- Sistema -->
        <div class="card">
            <div class="card-header">
                <h3 class="card-title">🖥️ Sistema</h3>
                <div class="card-status" id="system-status">-</div>
            </div>
            
            <div class="metric">
                <div class="metric-label">
                    <span class="metric-name">CPU</span>
                    <span class="metric-value" id="cpu-value">-</span>
                </div>
                <div class="progress-bar">
                    <div class="progress-fill progress-cpu" id="cpu-bar" style="width: 0%"></div>
                </div>
            </div>
            
            <div class="metric">
                <div class="metric-label">
                    <span class="metric-name">Memória</span>
                    <span class="metric-value" id="memory-value">-</span>
                </div>
                <div class="progress-bar">
                    <div class="progress-fill progress-memory" id="memory-bar" style="width: 0%"></div>
                </div>
            </div>
            
            <div class="metric">
                <div class="metric-label">
                    <span class="metric-name">Disco (/opt)</span>
                    <span class="metric-value" id="disk-value">-</span>
                </div>
                <div class="progress-bar">
                    <div class="progress-fill progress-disk" id="disk-bar" style="width: 0%"></div>
                </div>
            </div>
            
            <div class="chart-container">
                <canvas id="system-chart"></canvas>
            </div>
        </div>
        
        <!-- PostgreSQL -->
        <div class="card">
            <div class="card-header">
                <h3 class="card-title">🐘 PostgreSQL</h3>
                <div class="card-status" id="postgresql-status">-</div>
            </div>
            
            <div class="stats-grid">
                <div class="stat-box">
                    <div class="stat-value" id="pg-connections">-</div>
                    <div class="stat-label">Conexões</div>
                </div>
                <div class="stat-box">
                    <div class="stat-value" id="pg-size">-</div>
                    <div class="stat-label">Tamanho (MB)</div>
                </div>
            </div>
            
            <div class="metric">
                <div class="metric-label">
                    <span class="metric-name">Queries Ativas</span>
                    <span class="metric-value" id="pg-queries">-</span>
                </div>
            </div>
            
            <div class="metric">
                <div class="metric-label">
                    <span class="metric-name">Locks em Espera</span>
                    <span class="metric-value" id="pg-locks">-</span>
                </div>
            </div>
        </div>
        
        <!-- Redis -->
        <div class="card">
            <div class="card-header">
                <h3 class="card-title">🔴 Redis</h3>
                <div class="card-status" id="redis-status">-</div>
            </div>
            
            <div class="metric">
                <div class="metric-label">
                    <span class="metric-name">Memória Usada</span>
                    <span class="metric-value" id="redis-memory">-</span>
                </div>
                <div class="progress-bar">
                    <div class="progress-fill progress-redis" id="redis-bar" style="width: 0%"></div>
                </div>
            </div>
            
            <div class="stats-grid">
                <div class="stat-box">
                    <div class="stat-value" id="redis-clients">-</div>
                    <div class="stat-label">Clientes</div>
                </div>
                <div class="stat-box">
                    <div class="stat-value" id="redis-hitrate">-</div>
                    <div class="stat-label">Hit Rate %</div>
                </div>
            </div>
        </div>
        
        <!-- Aplicação -->
        <div class="card">
            <div class="card-header">
                <h3 class="card-title">🚀 Aplicação</h3>
                <div class="card-status" id="app-status">-</div>
            </div>
            
            <div class="stats-grid">
                <div class="stat-box">
                    <div class="stat-value" id="app-ops">-</div>
                    <div class="stat-label">OPs Ativas</div>
                </div>
                <div class="stat-box">
                    <div class="stat-value" id="app-users">-</div>
                    <div class="stat-label">Usuários Ativos</div>
                </div>
                <div class="stat-box">
                    <div class="stat-value" id="app-pointings">-</div>
                    <div class="stat-label">Apontamentos Hoje</div>
                </div>
                <div class="stat-box">
                    <div class="stat-value" id="app-sync">-</div>
                    <div class="stat-label">Pendências Sync</div>
                </div>
            </div>
            
            <div class="chart-container">
                <canvas id="app-chart"></canvas>
            </div>
        </div>
        
        <!-- Produção (Tempo Real) -->
        <div class="card">
            <div class="card-header">
                <h3 class="card-title">🏭 Produção em Tempo Real</h3>
            </div>
            
            <div id="production-container">
                <!-- Dados de produção serão inseridos aqui -->
                <div style="text-align: center; padding: 20px;">
                    <div class="spinner"></div>
                    <p>Carregando dados de produção...</p>
                </div>
            </div>
        </div>
        
        <div class="refresh-info">
            Última atualização: <span id="last-update">-</span> |
            Próxima atualização em: <span id="next-update">-</span> segundos
        </div>
    </div>

    <script>
        // Configurações
        const UPDATE_INTERVAL = 10000; // 10 segundos
        let lastUpdateTime = null;
        let updateTimer = null;
        
        // Gráficos
        let systemChart = null;
        let appChart = null;
        
        // Dados históricos
        const systemHistory = {
            labels: [],
            cpu: [],
            memory: [],
            disk: []
        };
        
        const appHistory = {
            labels: [],
            ops: [],
            users: [],
            pointings: []
        };
        
        // Socket.io para atualização em tempo real
        const socket = io();
        
        // Conectar ao WebSocket
        socket.on('connect', () => {
            console.log('Conectado ao servidor WebSocket');
        });
        
        socket.on('status_update', (data) => {
            updateDashboard(data);
        });
        
        socket.on('production_update', (data) => {
            updateProduction(data);
        });
        
        socket.on('alert', (data) => {
            showAlert(data);
        });
        
        // Função para buscar dados
        async function fetchStatus() {
            try {
                const response = await fetch('/api/status/detailed');
                const data = await response.json();
                updateDashboard(data);
            } catch (error) {
                console.error('Erro ao buscar status:', error);
            }
        }
        
        // Função para atualizar dashboard
        function updateDashboard(data) {
            lastUpdateTime = new Date();
            
            // Status geral
            const overallStatus = data.overall;
            document.getElementById('overall-status').textContent = 
                `${overallStatus.status.toUpperCase()} (${overallStatus.health_percentage}%)`;
            document.getElementById('overall-status').className = 
                `card-status status-${overallStatus.status}`;
            
            // Sistema
            if (data.components.system) {
                const system = data.components.system;
                updateMetric('cpu', system.cpu?.percent);
                updateMetric('memory', system.memory?.percent);
                updateMetric('disk', system.disk?.percent);
                
                // Atualizar gráfico de sistema
                updateSystemChart(system);
            }
            
            // PostgreSQL
            if (data.components.postgresql) {
                const pg = data.components.postgresql;
                document.getElementById('postgresql-status').textContent = 
                    pg.status?.toUpperCase() || 'ERROR';
                document.getElementById('postgresql-status').className = 
                    `card-status status-${pg.status || 'danger'}`;
                
                document.getElementById('pg-connections').textContent = 
                    pg.connections?.active || '-';
                document.getElementById('pg-queries').textContent = 
                    pg.connections?.queries || '-';
                document.getElementById('pg-locks').textContent = 
                    pg.waiting_locks || '-';
                document.getElementById('pg-size').textContent = 
                    pg.database_size_mb ? Math.round(pg.database_size_mb) : '-';
            }
            
            // Redis
            if (data.components.redis) {
                const redis = data.components.redis;
                document.getElementById('redis-status').textContent = 
                    redis.status?.toUpperCase() || 'ERROR';
                document.getElementById('redis-status').className = 
                    `card-status status-${redis.status || 'danger'}`;
                
                document.getElementById('redis-memory').textContent = 
                    redis.used_memory_mb ? `${Math.round(redis.used_memory_mb)} MB` : '-';
                document.getElementById('redis-clients').textContent = 
                    redis.connected_clients || '-';
                document.getElementById('redis-hitrate').textContent = 
                    redis.hit_rate ? `${Math.round(redis.hit_rate)}%` : '-';
                
                // Barra de memória Redis (assumindo 2GB máximo)
                const redisPercent = redis.used_memory_mb ? (redis.used_memory_mb / 2048 * 100) : 0;
                document.getElementById('redis-bar').style.width = `${Math.min(redisPercent, 100)}%`;
            }
            
            // Aplicação
            if (data.components.application) {
                const app = data.components.application;
                document.getElementById('app-status').textContent = 
                    app.status?.toUpperCase() || 'ERROR';
                document.getElementById('app-status').className = 
                    `card-status status-${app.status || 'danger'}`;
                
                document.getElementById('app-ops').textContent = app.active_operations || '-';
                document.getElementById('app-users').textContent = app.active_users || '-';
                document.getElementById('app-pointings').textContent = app.today_pointings || '-';
                document.getElementById('app-sync').textContent = app.pending_sync || '-';
                
                // Atualizar gráfico de aplicação
                updateAppChart(app);
            }
            
            // Alertas
            updateAlerts(data.alerts || []);
            
            // Atualizar informações de atualização
            updateRefreshInfo();
        }
        
        // Função para atualizar métrica com barra de progresso
        function updateMetric(type, value) {
            if (value !== undefined) {
                const element = document.getElementById(`${type}-value`);
                const bar = document.getElementById(`${type}-bar`);
                
                element.textContent = `${Math.round(value)}%`;
                bar.style.width = `${value}%`;
                
                // Atualizar status do sistema
                let status = 'healthy';
                if (value > 85) status = 'danger';
                else if (value > 70) status = 'warning';
                
                document.getElementById('system-status').textContent = status.toUpperCase();
                document.getElementById('system-status').className = `card-status status-${status}`;
            }
        }
        
        // Função para atualizar gráfico de sistema
        function updateSystemChart(system) {
            const now = new Date().toLocaleTimeString();
            
            // Adicionar novos dados
            systemHistory.labels.push(now);
            systemHistory.cpu.push(system.cpu?.percent || 0);
            systemHistory.memory.push(system.memory?.percent || 0);
            systemHistory.disk.push(system.disk?.percent || 0);
            
            // Manter apenas últimos 20 pontos
            if (systemHistory.labels.length > 20) {
                systemHistory.labels.shift();
                systemHistory.cpu.shift();
                systemHistory.memory.shift();
                systemHistory.disk.shift();
            }
            
            // Criar ou atualizar gráfico
            const ctx = document.getElementById('system-chart').getContext('2d');
            
            if (!systemChart) {
                systemChart = new Chart(ctx, {
                    type: 'line',
                    data: {
                        labels: systemHistory.labels,
                        datasets: [
                            {
                                label: 'CPU',
                                data: systemHistory.cpu,
                                borderColor: '#3498db',
                                backgroundColor: 'rgba(52, 152, 219, 0.1)',
                                tension: 0.4
                            },
                            {
                                label: 'Memória',
                                data: systemHistory.memory,
                                borderColor: '#27ae60',
                                backgroundColor: 'rgba(39, 174, 96, 0.1)',
                                tension: 0.4
                            },
                            {
                                label: 'Disco',
                                data: systemHistory.disk,
                                borderColor: '#f39c12',
                                backgroundColor: 'rgba(243, 156, 18, 0.1)',
                                tension: 0.4
                            }
                        ]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        scales: {
                            y: {
                                beginAtZero: true,
                                max: 100,
                                ticks: {
                                    callback: function(value) {
                                        return value + '%';
                                    }
                                }
                            }
                        },
                        plugins: {
                            legend: {
                                position: 'top',
                            }
                        }
                    }
                });
            } else {
                systemChart.data.labels = systemHistory.labels;
                systemChart.data.datasets[0].data = systemHistory.cpu;
                systemChart.data.datasets[1].data = systemHistory.memory;
                systemChart.data.datasets[2].data = systemHistory.disk;
                systemChart.update('none');
            }
        }
        
        // Função para atualizar gráfico de aplicação
        function updateAppChart(app) {
            const now = new Date().toLocaleTimeString();
            
            // Adicionar novos dados
            appHistory.labels.push(now);
            appHistory.ops.push(app.active_operations || 0);
            appHistory.users.push(app.active_users || 0);
            appHistory.pointings.push(app.today_pointings || 0);
            
            // Manter apenas últimos 20 pontos
            if (appHistory.labels.length > 20) {
                appHistory.labels.shift();
                appHistory.ops.shift();
                appHistory.users.shift();
                appHistory.pointings.shift();
            }
            
            // Criar ou atualizar gráfico
            const ctx = document.getElementById('app-chart').getContext('2d');
            
            if (!appChart) {
                appChart = new Chart(ctx, {
                    type: 'line',
                    data: {
                        labels: appHistory.labels,
                        datasets: [
                            {
                                label: 'OPs Ativas',
                                data: appHistory.ops,
                                borderColor: '#9b59b6',
                                backgroundColor: 'rgba(155, 89, 182, 0.1)',
                                tension: 0.4
                            },
                            {
                                label: 'Usuários Ativos',
                                data: appHistory.users,
                                borderColor: '#1abc9c',
                                backgroundColor: 'rgba(26, 188, 156, 0.1)',
                                tension: 0.4
                            }
                        ]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        scales: {
                            y: {
                                beginAtZero: true,
                                ticks: {
                                    stepSize: 1
                                }
                            }
                        },
                        plugins: {
                            legend: {
                                position: 'top',
                            }
                        }
                    }
                });
            } else {
                appChart.data.labels = appHistory.labels;
                appChart.data.datasets[0].data = appHistory.ops;
                appChart.data.datasets[1].data = appHistory.users;
                appChart.update('none');
            }
        }
        
        // Função para atualizar alertas
        function updateAlerts(alerts) {
            const container = document.getElementById('alerts-container');
            container.innerHTML = '';
            
            if (alerts.length === 0) {
                container.innerHTML = `
                    <div class="alert" style="background-color: rgba(39, 174, 96, 0.1); border-left-color: #27ae60;">
                        <div class="alert-icon">✓</div>
                        <div>Todos os sistemas operando normalmente</div>
                    </div>
                `;
                return;
            }
            
            alerts.forEach(alert => {
                const alertClass = alert.level === 'warning' ? 'alert-warning' : 'alert-danger';
                const icon = alert.level === 'warning' ? '⚠️' : '🚨';
                
                const alertElement = document.createElement('div');
                alertElement.className = `alert ${alertClass}`;
                alertElement.innerHTML = `
                    <div class="alert-icon">${icon}</div>
                    <div>
                        <strong>${alert.component}</strong>: ${alert.message}
                    </div>
                `;
                
                container.appendChild(alertElement);
            });
        }
        
        // Função para mostrar alerta em tempo real
        function showAlert(alert) {
            const container = document.getElementById('alerts-container');
            
            const alertClass = alert.level === 'warning' ? 'alert-warning' : 'alert-danger';
            const icon = alert.level === 'warning' ? '⚠️' : '🚨';
            
            const alertElement = document.createElement('div');
            alertElement.className = `alert ${alertClass}`;
            alertElement.innerHTML = `
                <div class="alert-icon">${icon}</div>
                <div>
                    <strong>${alert.component}</strong>: ${alert.message}
                    <br><small>${new Date().toLocaleTimeString()}</small>
                </div>
            `;
            
            // Adicionar no início
            container.insertBefore(alertElement, container.firstChild);
            
            // Remover após 30 segundos
            setTimeout(() => {
                if (alertElement.parentNode) {
                    alertElement.remove();
                }
            }, 30000);
        }
        
        // Função para atualizar informações de atualização
        function updateRefreshInfo() {
            const now = new Date();
            document.getElementById('last-update').textContent = 
                now.toLocaleTimeString();
            
            if (updateTimer) {
                clearTimeout(updateTimer);
            }
            
            let secondsLeft = UPDATE_INTERVAL / 1000;
            const updateCountdown = () => {
                document.getElementById('next-update').textContent = secondsLeft;
                secondsLeft--;
                
                if (secondsLeft >= 0) {
                    setTimeout(updateCountdown, 1000);
                }
            };
            
            updateCountdown();
            
            // Agendar próxima atualização
            updateTimer = setTimeout(fetchStatus, UPDATE_INTERVAL);
        }
        
        // Função para atualizar dados de produção
        async function updateProduction(data) {
            const container = document.getElementById('production-container');
            
            if (!data || data.length === 0) {
                container.innerHTML = `
                    <div style="text-align: center; padding: 20px; color: #999;">
                        Nenhuma produção ativa no momento
                    </div>
                `;
                return;
            }
            
            let html = '<div class="production-list">';
            
            data.forEach(machine => {
                const statusColor = machine.status === 'working' ? '#27ae60' : 
                                  machine.status === 'stopped' ? '#e74c3c' : 
                                  machine.status === 'setup' ? '#f39c12' : '#95a5a6';
                
                const statusText = machine.status === 'working' ? 'Produzindo' :
                                 machine.status === 'stopped' ? 'Parada' :
                                 machine.status === 'setup' ? 'Setup' : 'Outro';
                
                html += `
                    <div class="machine-item" style="
                        padding: 10px;
                        margin: 5px 0;
                        border-left: 4px solid ${statusColor};
                        background-color: white;
                        border-radius: 4px;
                    ">
                        <div style="display: flex; justify-content: space-between; align-items: center;">
                            <div>
                                <strong>${machine.machine_name}</strong>
                                <div style="font-size: 0.8rem; color: #666;">
                                    ${machine.machine_code} • ${machine.department}
                                </div>
                            </div>
                            <div style="
                                padding: 4px 8px;
                                border-radius: 12px;
                                background-color: ${statusColor}20;
                                color: ${statusColor};
                                font-size: 0.8rem;
                                font-weight: 600;
                            ">
                                ${statusText}
                            </div>
                        </div>
                        
                        ${machine.op_number ? `
                        <div style="margin-top: 8px; padding: 8px; background-color: #f8f9fa; border-radius: 4px;">
                            <div><strong>OP:</strong> ${machine.op_number}</div>
                            <div><strong>Produto:</strong> ${machine.product_description || 'N/A'}</div>
                            <div><strong>Operador:</strong> ${machine.operator_name || 'N/A'}</div>
                            <div><strong>Tempo:</strong> ${Math.round(machine.current_duration_minutes)} minutos</div>
                        </div>
                        ` : ''}
                    </div>
                `;
            });
            
            html += '</div>';
            container.innerHTML = html;
        }
        
        // Buscar dados iniciais
        fetchStatus();
        
        // Buscar dados de produção inicial
        fetch('/api/production/current')
            .then(response => response.json())
            .then(data => updateProduction(data))
            .catch(error => console.error('Erro ao buscar produção:', error));
        
        // Configurar atualização automática
        setInterval(() => {
            fetch('/api/production/current')
                .then(response => response.json())
                .then(data => updateProduction(data))
                .catch(error => console.error('Erro ao buscar produção:', error));
        }, 30000); // A cada 30 segundos
        
        // Limpar timer ao sair da página
        window.addEventListener('beforeunload', () => {
            if (updateTimer) {
                clearTimeout(updateTimer);
            }
            socket.disconnect();
        });
    </script>
</body>
</html>
```

---

## **12. EXPANSÕES FUTURAS**

### **12.1 Roadmap de Desenvolvimento**

#### **Fase 1: MVP (Meses 1-3)**
- [ ] Sistema básico de apontamento
- [ ] Cadastro de máquinas e produtos
- [ ] Integração com ERP para OPs
- [ ] Relatórios básicos
- [ ] Autenticação de operadores

#### **Fase 2: Funcionalidades Core (Meses 4-6)**
- [ ] Gestão completa de paradas
- [ ] Cálculo automático de eficiência
- [ ] Painéis de controle em tempo real
- [ ] Módulo de qualidade integrado
- [ ] Notificações e alertas

#### **Fase 3: Otimização LEAN (Meses 7-9)**
- [ ] OEE completo por máquina
- [ ] Análise de perdas (6 grandes perdas)
- [ ] Kanban digital
- [ ] SMED (troca rápida de ferramentas)
- [ ] Poka-yoke digital

#### **Fase 4: Expansão e Integração (Meses 10-12)**
- [ ] Integração com sensores IoT (ESP32)
- [ ] PCP automático
- [ ] Manutenção preditiva
- [ ] Módulo de competências
- [ ] Integração com BI

### **12.2 Integração com ESP32/IoT**

#### **12.2.1 Arquitetura de Integração IoT**
```
┌─────────────────┐    MQTT/HTTP    ┌─────────────────┐
│   ESP32/Sensor  ├─────────────────▶│   Broker MQTT   │
│                 │                 │  (Mosquitto)    │
│ • Encoder       │                 └────────┬────────┘
│ • Sensor óptico │                          │
│ • Temperatura   │                 ┌────────▼────────┐
│ • Vibração      │                 │   API Gateway   │
└─────────────────┘                 │    (Flask)      │
                                    └────────┬────────┘
                                             │
                                    ┌────────▼────────┐
                                    │   Production    │
                                    │    Pointer      │
                                    └─────────────────┘
```

#### **12.2.2 Endpoints para IoT**
```python
# app/api/iot.py
from flask import Blueprint, request, jsonify
from datetime import datetime
import json

iot_bp = Blueprint('iot', __name__)

@iot_bp.route('/api/iot/data', methods=['POST'])
def receive_iot_data():
    """Receber dados de sensores IoT"""
    try:
        data = request.get_json()
        
        # Validar dados
        required_fields = ['device_id', 'timestamp', 'sensor_type', 'value']
        for field in required_fields:
            if field not in data:
                return jsonify({'error': f'Missing field: {field}'}), 400
        
        # Registrar dados
        iot_reading = IoTReading(
            device_id=data['device_id'],
            machine_code=data.get('machine_code'),
            sensor_type=data['sensor_type'],
            value=data['value'],
            unit=data.get('unit'),
            timestamp=datetime.fromisoformat(data['timestamp']),
            received_at=datetime.utcnow()
        )
        
        db.session.add(iot_reading)
        db.session.commit()
        
        # Processar dados em tempo real
        process_iot_data(iot_reading)
        
        return jsonify({'status': 'success', 'id': iot_reading.id}), 201
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500

def process_iot_data(reading):
    """Processar dados IoT em tempo real"""
    
    if reading.sensor_type == 'encoder':
        # Calcular velocidade em tempo real
        machine = Machine.query.filter_by(code=reading.machine_code).first()
        if machine and machine.current_op:
            # Atualizar velocidade atual da máquina
            machine.current_speed = reading.value
            
            # Alertar se fora dos parâmetros
            speed_params = get_speed_parameters(machine.current_op.product_code)
            if reading.value < speed_params['min_speed']:
                create_alert(
                    type='low_speed',
                    machine_id=machine.id,
                    value=reading.value,
                    threshold=speed_params['min_speed']
                )
    
    elif reading.sensor_type == 'temperature':
        # Monitorar temperatura de secagem
        if reading.value > 120:  # °C
            create_alert(
                type='high_temperature',
                machine_id=machine.id,
                value=reading.value,
                threshold=120
            )
    
    elif reading.sensor_type == 'vibration':
        # Detectar vibração anormal
        if reading.value > 5.0:  # m/s²
            create_alert(
                type='high_vibration',
                machine_id=machine.id,
                value=reading.value,
                threshold=5.0
            )
```

### **12.3 Módulo de PCP Automático**

#### **12.3.1 Funcionalidades do PCP**
```python
# modules/pcp/planner.py
class ProductionPlanner:
    
    def __init__(self):
        self.sales_orders = []
        self.current_stock = {}
        self.production_capacity = {}
        self.lead_times = {}
    
    def load_data(self):
        """Carregar dados do ERP"""
        # Pedidos de venda
        self.sales_orders = self.get_sales_orders()
        
        # Estoque atual
        self.current_stock = self.get_current_stock()
        
        # Capacidade de produção
        self.production_capacity = self.get_production_capacity()
        
        # Lead times
        self.lead_times = self.get_lead_times()
    
    def calculate_mrp(self):
        """Calcular necessidades de materiais"""
        mrp_results = []
        
        for product_code, requirements in self.calculate_requirements():
            # Verificar estoque atual
            current_stock = self.current_stock.get(product_code, 0)
            safety_stock = self.get_safety_stock(product_code)
            
            # Calcular necessidade líquida
            net_requirement = max(0, requirements - current_stock + safety_stock)
            
            if net_requirement > 0:
                # Calcular quando produzir
                production_date = self.calculate_production_date(
                    product_code, 
                    net_requirement
                )
                
                mrp_results.append({
                    'product_code': product_code,
                    'requirement': requirements,
                    'current_stock': current_stock,
                    'net_requirement': net_requirement,
                    'production_date': production_date,
                    'priority': self.calculate_priority(product_code)
                })
        
        return mrp_results
    
    def generate_production_schedule(self):
        """Gerar programação de produção"""
        schedule = {
            'daily': {},
            'weekly': {},
            'monthly': {}
        }
        
        # Agendar por máquina
        for machine in self.get_available_machines():
            machine_schedule = self.schedule_for_machine(machine)
            schedule['daily'][machine.code] = machine_schedule
        
        # Otimizar sequenciamento
        optimized_schedule = self.optimize_sequence(schedule)
        
        return optimized_schedule
    
    def optimize_sequence(self, schedule):
        """Otimizar sequenciamento usando algoritmos"""
        # Implementar:
        # 1. SPT (Shortest Processing Time)
        # 2. EDD (Earliest Due Date)
        # 3. Johnson's rule (para 2 máquinas)
        # 4. Algoritmo genético para sequenciamento complexo
        pass
    
    def simulate_scenarios(self, scenario_params):
        """Simular diferentes cenários de produção"""
        scenarios = []
        
        # Cenário 1: Aumento de demanda
        scenarios.append(self.simulate_demand_increase(scenario_params))
        
        # Cenário 2: Quebra de máquina
        scenarios.append(self.simulate_machine_breakdown(scenario_params))
        
        # Cenário 3: Atraso de material
        scenarios.append(self.simulate_material_delay(scenario_params))
        
        # Analisar impactos
        analysis = self.analyze_scenarios(scenarios)
        
        return {
            'scenarios': scenarios,
            'analysis': analysis,
            'recommendations': self.generate_recommendations(analysis)
        }
```

### **12.4 Módulo de Business Intelligence**

#### **12.4.1 Data Warehouse para Análises**
```sql
-- Schema do Data Warehouse
CREATE SCHEMA dw_production;

-- Dimensão Tempo
CREATE TABLE dw_production.dim_time (
    time_key SERIAL PRIMARY KEY,
    full_date DATE NOT NULL,
    year INTEGER NOT NULL,
    quarter INTEGER NOT NULL,
    month INTEGER NOT NULL,
    week INTEGER NOT NULL,
    day_of_week INTEGER NOT NULL,
    is_weekday BOOLEAN NOT NULL,
    is_holiday BOOLEAN NOT NULL
);

-- Dimensão Máquina
CREATE TABLE dw_production.dim_machine (
    machine_key SERIAL PRIMARY KEY,
    machine_code VARCHAR(20) NOT NULL,
    machine_name VARCHAR(100) NOT NULL,
    machine_type VARCHAR(50) NOT NULL,
    department VARCHAR(50) NOT NULL,
    manufacturer VARCHAR(100),
    installation_date DATE,
    age_years INTEGER
);

-- Dimensão Produto
CREATE TABLE dw_production.dim_product (
    product_key SERIAL PRIMARY KEY,
    product_code VARCHAR(20) NOT NULL,
    product_description VARCHAR(200) NOT NULL,
    fabric_type VARCHAR(50),
    fabric_weight DECIMAL(6,2),
    fabric_width DECIMAL(6,2),
    color_type VARCHAR(20)
);

-- Dimensão Operador
CREATE TABLE dw_production.dim_operator (
    operator_key SERIAL PRIMARY KEY,
    employee_code VARCHAR(20) NOT NULL,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    position VARCHAR(50),
    experience_months INTEGER,
    qualification_level VARCHAR(20)
);

-- Fatos de Produção
CREATE TABLE dw_production.fact_production (
    production_key SERIAL PRIMARY KEY,
    time_key INTEGER REFERENCES dw_production.dim_time(time_key),
    machine_key INTEGER REFERENCES dw_production.dim_machine(machine_key),
    product_key INTEGER REFERENCES dw_production.dim_product(product_key),
    operator_key INTEGER REFERENCES dw_production.dim_operator(operator_key),
    
    -- Métricas
    produced_quantity DECIMAL(10,2),
    good_quantity DECIMAL(10,2),
    rejected_quantity DECIMAL(10,2),
    production_minutes INTEGER,
    stop_minutes INTEGER,
    setup_minutes INTEGER,
    
    -- KPIs
    availability DECIMAL(5,2),
    performance DECIMAL(5,2),
    quality_rate DECIMAL(5,2),
    oee DECIMAL(5,2),
    efficiency DECIMAL(5,2),
    
    -- Custos
    material_cost DECIMAL(10,2),
    labor_cost DECIMAL(10,2),
    energy_cost DECIMAL(10,2),
    total_cost DECIMAL(10,2),
    cost_per_unit DECIMAL(10,4)
);

-- ETL Process
CREATE OR REPLACE PROCEDURE dw_production.refresh_data()
LANGUAGE plpgsql
AS $$
BEGIN
    -- 1. Limpar dados antigos
    TRUNCATE TABLE dw_production.fact_production;
    
    -- 2. Carregar dimensões
    -- Tempo
    INSERT INTO dw_production.dim_time (...)
    SELECT ... FROM generate_series(...);
    
    -- Máquinas
    INSERT INTO dw_production.dim_machine (...)
    SELECT ... FROM production.machines;
    
    -- 3. Carregar fatos
    INSERT INTO dw_production.fact_production (...)
    SELECT 
        dt.time_key,
        dm.machine_key,
        dp.product_key,
        do.operator_key,
        pp.produced_quantity,
        -- ... outros campos
    FROM production.production_pointing pp
    JOIN dw_production.dim_time dt ON DATE(pp.start_time) = dt.full_date
    JOIN dw_production.dim_machine dm ON pp.machine_id = dm.machine_id
    -- ... outros joins
    
    -- 4. Atualizar agregados
    REFRESH MATERIALIZED VIEW dw_production.mv_daily_summary;
    REFRESH MATERIALIZED VIEW dw_production.mv_weekly_trends;
    
    COMMIT;
END;
$$;
```

### **12.5 Considerações Finais**

#### **12.5.1 Lições Aprendidas com Implantação**
1. **Engajamento do usuário é crítico**: Envolver operadores desde o início
2. **Dados limpos são fundamentais**: Investir tempo na qualidade dos dados iniciais
3. **Wi-Fi industrial robusto**: Não subestimar a infraestrutura de rede
4. **Treinamento contínuo**: Sistemas evoluem, usuários precisam acompanhar
5. **Integração gradual**: Começar com módulos simples, depois expandir

#### **12.5.2 Métricas de Sucesso**
```yaml
sucesso_curto_prazo:
  - Redução de 50% no tempo de apontamento
  - Eliminação de 100% das planilhas Excel
  - Aumento de 10% na cobertura de apontamento
  
sucesso_medio_prazo:
  - Aumento de 5% no OEE geral
  - Redução de 20% no tempo de parada
  - Melhoria de 15% na previsibilidade
  
sucesso_longo_prazo:
  - Integração completa com PCP
  - Implementação de manutenção preditiva
  - Redução de 30% nos custos de produção
```

#### **12.5.3 Sustentabilidade do Sistema**
1. **Documentação viva**: Manter documentação atualizada com o sistema
2. **Governança de dados**: Políticas claras para qualidade e segurança
3. **Roadmap tecnológico**: Planejar upgrades e novas tecnologias
4. **Comunidade de prática**: Envolver usuários na melhoria contínua
5. **Indicadores de saúde**: Monitorar adoção e satisfação

---

**Este documento representa um guia completo para implantação, operação e evolução do sistema Production Pointer Pro em ambiente de servidor físico dedicado. A abordagem foi planejada para garantir robustez, segurança e escalabilidade, atendendo às necessidades específicas da indústria têxtil com foco na metodologia LEAN.**