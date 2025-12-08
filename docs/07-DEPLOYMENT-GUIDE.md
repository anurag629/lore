# Deployment Guide

## Overview

This guide covers setting up the **Lore** platform for local development and production deployment. The platform consists of:
- **Backend:** Django 5.1 + PostgreSQL + Redis
- **Frontend:** Next.js 16 + React 19
- **Blockchain:** Story Protocol (Aeneid Testnet)
- **AI:** LiteLLM + OpenRouter
- **Storage:** Pinata IPFS

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Local Development Setup](#local-development-setup)
3. [Environment Configuration](#environment-configuration)
4. [Database Setup](#database-setup)
5. [Running the Application](#running-the-application)
6. [Production Deployment](#production-deployment)
7. [Monitoring & Maintenance](#monitoring--maintenance)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| Python | 3.11+ | Backend runtime |
| Node.js | 18.x+ | Frontend runtime |
| PostgreSQL | 15.x+ | Database |
| Redis | 7.0+ | Caching |
| Git | 2.40+ | Version control |

### Required Accounts

1. **Reown (formerly WalletConnect)** - https://cloud.reown.com
   - Project ID for wallet integration
   - Free tier available

2. **OpenRouter** - https://openrouter.ai
   - API key for AI models
   - Free tier (Gemini Flash, Llama 3.2, Mistral)

3. **Pinata** - https://www.pinata.cloud
   - JWT token for IPFS uploads
   - Free tier: 1GB storage, 100GB bandwidth/month

4. **Story Protocol** - https://faucet.story.foundation
   - Testnet wallet with IP tokens
   - Private key for blockchain transactions

### System Requirements

**Development Machine:**
- CPU: 2+ cores
- RAM: 8GB+ recommended
- Disk: 10GB+ free space
- OS: Windows 10/11, macOS 10.15+, Linux (Ubuntu 20.04+)

**Production Server:**
- CPU: 4+ cores
- RAM: 16GB+ recommended
- Disk: 50GB+ SSD
- OS: Ubuntu 22.04 LTS recommended

---

## Local Development Setup

### Step 1: Clone Repository

```bash
git clone https://github.com/yourusername/lore-platform.git
cd lore-platform
```

### Step 2: Backend Setup

#### 2.1 Create Virtual Environment

**Windows:**
```bash
cd lore_backend
python -m venv .venv
.\.venv\Scripts\activate
```

**macOS/Linux:**
```bash
cd lore_backend
python3 -m venv .venv
source .venv/bin/activate
```

#### 2.2 Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements/development.txt
```

**Expected Output:**
```
Successfully installed Django-5.1.0 djangorestframework-3.14.0 ...
```

#### 2.3 Create Environment File

```bash
cp .env.example .env
```

Edit `.env` with your credentials (see [Environment Configuration](#environment-configuration))

#### 2.4 Install PostgreSQL

**Windows (using Chocolatey):**
```bash
choco install postgresql
```

**macOS (using Homebrew):**
```bash
brew install postgresql@15
brew services start postgresql@15
```

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
```

#### 2.5 Create Database

```bash
# Connect to PostgreSQL
psql -U postgres

# Create database and user
CREATE DATABASE lore_db;
CREATE USER lore_user WITH PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE lore_db TO lore_user;
\q
```

#### 2.6 Install Redis

**Windows (using Chocolatey):**
```bash
choco install redis-64
redis-server
```

**macOS (using Homebrew):**
```bash
brew install redis
brew services start redis
```

**Ubuntu/Debian:**
```bash
sudo apt install redis-server
sudo systemctl start redis-server
```

**Verify Redis:**
```bash
redis-cli ping
# Expected output: PONG
```

#### 2.7 Run Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

**Expected Output:**
```
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying users.0001_initial... OK
  Applying assets.0001_initial... OK
  ...
```

#### 2.8 Create Superuser (Optional)

```bash
python manage.py createsuperuser --wallet_address 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb --username admin
```

### Step 3: Frontend Setup

#### 3.1 Navigate to Frontend Directory

```bash
cd ../lore-frontend
```

#### 3.2 Install Dependencies

```bash
npm install
```

**Expected Output:**
```
added 1243 packages in 45s
```

#### 3.3 Create Environment File

```bash
cp .env.example .env.local
```

Edit `.env.local` with your configuration:
```bash
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_REOWN_PROJECT_ID=your_reown_project_id
```

---

## Environment Configuration

### Backend Environment Variables (.env)

```bash
# ====================
# Django Settings
# ====================
SECRET_KEY=django-insecure-CHANGE-THIS-IN-PRODUCTION-abc123def456ghi789
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# ====================
# Database (PostgreSQL)
# ====================
DB_NAME=lore_db
DB_USER=lore_user
DB_PASSWORD=your_secure_password
DB_HOST=localhost
DB_PORT=5432
DATABASE_URL=postgresql://lore_user:your_secure_password@localhost:5432/lore_db

# ====================
# Redis Cache
# ====================
REDIS_URL=redis://localhost:6379/0

# ====================
# Story Protocol (Testnet)
# ====================
STORY_NETWORK=testnet
STORY_RPC_URL=https://aeneid.storyrpc.io/
STORY_PRIVATE_KEY=0xYOUR_PRIVATE_KEY_HERE
NFT_CONTRACT_ADDRESS=0xYOUR_NFT_CONTRACT_ADDRESS

# ====================
# Pinata IPFS
# ====================
PINATA_JWT=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.YOUR_JWT_HERE

# ====================
# OpenRouter AI
# ====================
OPENROUTER_API_KEY=sk-or-v1-YOUR_API_KEY_HERE

# AI Model Configuration
DEFAULT_AI_MODEL=google/gemini-flash-1.5
AI_MAX_TOKENS=500
AI_TEMPERATURE=0.7
AI_REQUEST_TIMEOUT=30
AI_CACHE_ENABLED=True
AI_CACHE_TTL=3600

# ====================
# CORS (Frontend URL)
# ====================
CORS_ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
```

### Frontend Environment Variables (.env.local)

```bash
# ====================
# API Configuration
# ====================
NEXT_PUBLIC_API_URL=http://localhost:8000

# ====================
# Reown (WalletConnect)
# ====================
NEXT_PUBLIC_REOWN_PROJECT_ID=your_reown_project_id_from_cloud_reown_com

# ====================
# Blockchain Network
# ====================
NEXT_PUBLIC_CHAIN_ID=1315
NEXT_PUBLIC_CHAIN_NAME=Aeneid Testnet
NEXT_PUBLIC_RPC_URL=https://aeneid.storyrpc.io/
NEXT_PUBLIC_EXPLORER_URL=https://aeneid.storyscan.xyz/
```

### How to Get API Keys

#### 1. Reown Project ID

1. Go to https://cloud.reown.com
2. Sign up / Log in
3. Create new project: "Lore Platform"
4. Copy Project ID from dashboard
5. Paste into `NEXT_PUBLIC_REOWN_PROJECT_ID`

#### 2. OpenRouter API Key

1. Go to https://openrouter.ai
2. Sign up with email
3. Navigate to "Keys" section
4. Create new API key
5. Copy key (starts with `sk-or-v1-`)
6. Paste into `OPENROUTER_API_KEY`

**Note:** Free tier includes Gemini Flash, Llama 3.2, Mistral 7B

#### 3. Pinata JWT Token

1. Go to https://www.pinata.cloud
2. Sign up / Log in
3. Navigate to "API Keys"
4. Create new JWT key with permissions:
   - ✅ pinFileToIPFS
   - ✅ pinJSONToIPFS
5. Copy JWT token
6. Paste into `PINATA_JWT`

#### 4. Story Protocol Wallet

1. Create new Ethereum wallet (MetaMask recommended)
2. Export private key (Settings → Security → Export Private Key)
3. Go to https://faucet.story.foundation
4. Request testnet IP tokens
5. Copy private key (starts with `0x`)
6. Paste into `STORY_PRIVATE_KEY`

**Security Warning:** Never commit private keys to Git. Always use `.env` files and add to `.gitignore`.

---

## Database Setup

### Initial Migrations

```bash
cd lore_backend
python manage.py makemigrations users
python manage.py makemigrations assets
python manage.py migrate
```

### Load Fixtures (Optional Sample Data)

```bash
python manage.py loaddata fixtures/sample_users.json
python manage.py loaddata fixtures/sample_assets.json
```

### Database Backup

**Backup:**
```bash
pg_dump -U lore_user lore_db > backup_$(date +%Y%m%d).sql
```

**Restore:**
```bash
psql -U lore_user lore_db < backup_20241204.sql
```

---

## Running the Application

### Start Backend Server

```bash
cd lore_backend
python manage.py runserver 8000
```

**Expected Output:**
```
Django version 5.1.0, using settings 'config.settings.development'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

**Verify Backend:**
- Admin Panel: http://localhost:8000/admin
- API Root: http://localhost:8000/api/

### Start Frontend Server

Open a new terminal:

```bash
cd lore-frontend
npm run dev
```

**Expected Output:**
```
  ▲ Next.js 16.0.0
  - Local:        http://localhost:3000
  - Network:      http://192.168.1.100:3000

 ✓ Ready in 2.3s
```

**Verify Frontend:**
- Landing Page: http://localhost:3000
- Dashboard: http://localhost:3000/dashboard

### Verify Services

**Check PostgreSQL:**
```bash
psql -U lore_user -d lore_db -c "SELECT COUNT(*) FROM users_user;"
```

**Check Redis:**
```bash
redis-cli
> PING
PONG
> KEYS *
(empty array or existing keys)
> EXIT
```

---

## Production Deployment

### Option 1: Azure + Vercel (Recommended)

#### Backend: Azure App Service

**Step 1: Prepare for Production**

1. Update `config/settings/production.py`:
```python
DEBUG = False
ALLOWED_HOSTS = ['your-app.azurewebsites.net']
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

2. Install production dependencies:
```bash
pip install gunicorn whitenoise
```

3. Create `Procfile`:
```
web: gunicorn config.wsgi:application --bind 0.0.0.0:$PORT
```

4. Freeze dependencies:
```bash
pip freeze > requirements.txt
```

**Step 2: Create Azure Resources**

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login
az login

# Create resource group
az group create --name lore-rg --location eastus

# Create App Service Plan
az appservice plan create \
  --name lore-plan \
  --resource-group lore-rg \
  --sku B1 \
  --is-linux

# Create Web App
az webapp create \
  --resource-group lore-rg \
  --plan lore-plan \
  --name lore-backend \
  --runtime "PYTHON:3.11"

# Create PostgreSQL Database
az postgres flexible-server create \
  --resource-group lore-rg \
  --name lore-db-server \
  --location eastus \
  --admin-user loreadmin \
  --admin-password YOUR_PASSWORD \
  --sku-name Standard_B1ms \
  --storage-size 32

# Create Redis Cache
az redis create \
  --resource-group lore-rg \
  --name lore-redis \
  --location eastus \
  --sku Basic \
  --vm-size c0
```

**Step 3: Configure Environment Variables**

```bash
az webapp config appsettings set \
  --resource-group lore-rg \
  --name lore-backend \
  --settings \
    SECRET_KEY="your-production-secret-key" \
    DEBUG=False \
    DATABASE_URL="postgresql://..." \
    REDIS_URL="redis://..." \
    OPENROUTER_API_KEY="sk-or-..." \
    PINATA_JWT="eyJhbG..." \
    STORY_PRIVATE_KEY="0x..."
```

**Step 4: Deploy Code**

```bash
# Deploy from Git
az webapp deployment source config \
  --resource-group lore-rg \
  --name lore-backend \
  --repo-url https://github.com/yourusername/lore-platform \
  --branch main \
  --manual-integration
```

#### Frontend: Vercel

**Step 1: Install Vercel CLI**

```bash
npm install -g vercel
```

**Step 2: Deploy**

```bash
cd lore-frontend
vercel login
vercel --prod
```

**Step 3: Configure Environment Variables**

In Vercel Dashboard:
1. Go to Project Settings → Environment Variables
2. Add:
   - `NEXT_PUBLIC_API_URL`: https://lore-backend.azurewebsites.net
   - `NEXT_PUBLIC_REOWN_PROJECT_ID`: your_project_id

**Step 4: Redeploy**

```bash
vercel --prod
```

---

### Option 2: Docker Deployment

#### Create Dockerfiles

**Backend Dockerfile:**
```dockerfile
# lore_backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy project
COPY . .

# Run migrations and start server
CMD python manage.py migrate && \
    gunicorn config.wsgi:application --bind 0.0.0.0:8000
```

**Frontend Dockerfile:**
```dockerfile
# lore-frontend/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json

EXPOSE 3000
CMD ["npm", "start"]
```

#### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: lore_db
      POSTGRES_USER: lore_user
      POSTGRES_PASSWORD: lore_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  backend:
    build: ./lore_backend
    command: gunicorn config.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - ./lore_backend:/app
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://lore_user:lore_password@db:5432/lore_db
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

  frontend:
    build: ./lore-frontend
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8000
    depends_on:
      - backend

volumes:
  postgres_data:
```

#### Run with Docker Compose

```bash
docker-compose up -d
```

---

## Monitoring & Maintenance

### Backend Monitoring

#### Django Admin

Access: http://localhost:8000/admin

**Monitor:**
- AI Generation Logs
- User Activity
- IP Assets
- Error Logs

#### Database Performance

```sql
-- Find slow queries
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Check table sizes
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

#### Redis Monitoring

```bash
redis-cli INFO stats
redis-cli INFO memory
```

### Log Files

**Django Logs:**
```bash
tail -f lore_backend/logs/django.log
```

**Next.js Logs:**
```bash
npm run dev  # Shows logs in terminal
```

### Backup Strategy

**Daily Database Backup (Cron Job):**
```bash
# Add to crontab (crontab -e)
0 2 * * * pg_dump -U lore_user lore_db > /backups/lore_db_$(date +\%Y\%m\%d).sql
```

**Backup Retention:**
- Keep daily backups for 7 days
- Keep weekly backups for 4 weeks
- Keep monthly backups for 6 months

---

## Troubleshooting

### Common Issues

#### 1. Database Connection Error

**Error:**
```
django.db.utils.OperationalError: could not connect to server
```

**Solution:**
```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Restart PostgreSQL
sudo systemctl restart postgresql

# Verify credentials in .env
DB_HOST=localhost
DB_PORT=5432
DB_USER=lore_user
DB_PASSWORD=correct_password
```

#### 2. Redis Connection Error

**Error:**
```
redis.exceptions.ConnectionError: Error connecting to Redis
```

**Solution:**
```bash
# Check Redis status
sudo systemctl status redis-server

# Restart Redis
sudo systemctl restart redis-server

# Test connection
redis-cli ping
```

#### 3. AI Model Rate Limit

**Error:**
```
litellm.RateLimitError: Rate limit exceeded for google/gemini-flash-1.5
```

**Solution:**
- Model fallback automatically tries next model
- Wait 60 seconds for rate limit reset
- Use cache (95% hit rate prevents this)

**Check Cache:**
```bash
redis-cli
> KEYS ai_cache:*
> GET ai_cache:abc123...
```

#### 4. IPFS Upload Failed

**Error:**
```
Failed to upload to Pinata: 401 Unauthorized
```

**Solution:**
```bash
# Verify JWT token
curl -X GET https://api.pinata.cloud/data/testAuthentication \
  -H "Authorization: Bearer YOUR_JWT"

# Should return: {"message":"Congratulations! You are communicating with the Pinata API!"}

# Update .env with correct JWT
PINATA_JWT=eyJhbGc...
```

#### 5. Blockchain Transaction Failed

**Error:**
```
Story Protocol transaction failed: insufficient funds
```

**Solution:**
1. Check wallet balance:
   ```bash
   # Visit: https://aeneid.storyscan.xyz/address/YOUR_WALLET
   ```

2. Request testnet tokens:
   ```bash
   # Visit: https://faucet.story.foundation
   ```

3. Wait for confirmation (10-30 seconds)

#### 6. Frontend Can't Connect to Backend

**Error:**
```
AxiosError: Network Error
```

**Solution:**
```bash
# Check backend is running
curl http://localhost:8000/api/

# Check CORS settings (backend .env)
CORS_ALLOWED_ORIGINS=http://localhost:3000

# Check frontend .env.local
NEXT_PUBLIC_API_URL=http://localhost:8000
```

#### 7. Wallet Connection Issues

**Error:**
```
Connector not found
```

**Solution:**
1. Install MetaMask browser extension
2. Clear browser cache
3. Check Reown Project ID in `.env.local`
4. Verify network configuration (Base Sepolia)

### Debug Mode

**Enable Django Debug Toolbar:**
```bash
pip install django-debug-toolbar
```

Add to `config/settings/development.py`:
```python
INSTALLED_APPS += ['debug_toolbar']
MIDDLEWARE += ['debug_toolbar.middleware.DebugToolbarMiddleware']
INTERNAL_IPS = ['127.0.0.1']
```

---

## Performance Optimization

### Backend Optimization

**1. Database Connection Pooling:**
```python
# config/settings/production.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'CONN_MAX_AGE': 600,  # Persistent connections
    }
}
```

**2. Query Optimization:**
```python
# Use select_related for foreign keys
assets = IPAsset.objects.select_related('user').all()

# Use prefetch_related for many-to-many
assets = IPAsset.objects.prefetch_related('derivatives').all()
```

**3. Redis Caching:**
```python
# Increase cache timeout for static data
cache.set('platform_stats', stats, timeout=3600)  # 1 hour
```

### Frontend Optimization

**1. Image Optimization:**
```typescript
import Image from 'next/image';

<Image
  src={asset.media_url}
  alt={asset.title}
  width={500}
  height={300}
  loading="lazy"
/>
```

**2. Code Splitting:**
```typescript
import dynamic from 'next/dynamic';

const MintModal = dynamic(() => import('@/components/mint/MintModal'), {
  loading: () => <Spinner />,
});
```

---

## Security Checklist

### Production Deployment

- [ ] Change `SECRET_KEY` to random 50-character string
- [ ] Set `DEBUG=False`
- [ ] Use HTTPS only (`SECURE_SSL_REDIRECT=True`)
- [ ] Enable CSRF protection
- [ ] Whitelist allowed hosts
- [ ] Use strong database passwords
- [ ] Rotate API keys regularly
- [ ] Enable rate limiting
- [ ] Set up firewall rules
- [ ] Regular security updates
- [ ] Monitor error logs
- [ ] Backup database daily

---

**Document Version:** 1.0
**Last Updated:** December 2024
**Next:** [08-FUTURE-ENHANCEMENTS.md](./08-FUTURE-ENHANCEMENTS.md)
