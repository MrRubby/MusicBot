# 🚀 TurkishArmy CS2 - Production Deployment Rehberi

Bu rehber, TurkishArmy CS2 web sitesini ve backend API'sini production ortamına deploy etmek için gereken tüm adımları içerir.

---

## 📋 İçindekiler

1. [Genel Bakış](#genel-bakış)
2. [Gereksinimler](#gereksinimler)
3. [Backend Deployment](#backend-deployment)
4. [Nginx Yapılandırması](#nginx-yapılandırması)
5. [Frontend Deployment](#frontend-deployment)
6. [DNS Ayarları](#dns-ayarları)
7. [SSL Sertifikası](#ssl-sertifikası)
8. [Test ve Doğrulama](#test-ve-doğrulama)
9. [Troubleshooting](#troubleshooting)

---

## 🎯 Genel Bakış

### Domain Yapısı
- **Frontend:** `https://turkisharmycs2.com`
- **Backend API:** `https://api.turkisharmycs2.com`

### Teknoloji Stack
- **Frontend:** React + Vite (Static hosting)
- **Backend:** Node.js + Express (Linux server)
- **Web Server:** Nginx (Reverse proxy)
- **Process Manager:** PM2
- **SSL:** Let's Encrypt (Certbot)

---

## 📦 Gereksinimler

### Linux Server (Backend için)
- Ubuntu 20.04+ veya Debian 11+
- Node.js 18+ ve npm
- Nginx
- PM2 (Process Manager)
- Certbot (SSL için)
- En az 1GB RAM
- En az 10GB disk alanı

### Kurulum Komutları

```bash
# Node.js 18.x kurulumu
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Nginx kurulumu
sudo apt-get update
sudo apt-get install -y nginx

# PM2 kurulumu
sudo npm install -g pm2

# Certbot kurulumu (SSL için)
sudo apt-get install -y certbot python3-certbot-nginx

# Git kurulumu
sudo apt-get install -y git
```

---

## 🔧 Backend Deployment

### 1. Projeyi Sunucuya Klonlayın

```bash
# Home dizinine gidin
cd ~

# Projeyi klonlayın
git clone https://github.com/yourusername/TurkishArmy-Websites.git
cd TurkishArmy-Websites/turkisharmy-cs2-backend
```

### 2. Environment Variables Ayarlayın

`.env` dosyası oluşturun:

```bash
nano .env
```

Aşağıdaki içeriği ekleyin:

```env
# Environment
NODE_ENV=production

# Server Ports
PORT=3001

# URLs
FRONTEND_URL=https://turkisharmycs2.com
BACKEND_URL=https://api.turkisharmycs2.com

# Steam API
STEAM_API_KEY=your_steam_api_key_here
STEAM_GROUP_ID=103582791429521408

# Session Security
SESSION_SECRET=your_very_secure_random_secret_here_min_32_chars
```

**Önemli Notlar:**
- `STEAM_API_KEY`: [Steam Web API Key](https://steamcommunity.com/dev/apikey) sayfasından alın
- `SESSION_SECRET`: Güçlü bir random string kullanın (en az 32 karakter)
  ```bash
  # Random secret oluşturmak için:
  node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
  ```

### 3. Dependencies Yükleyin

```bash
npm install
```

### 4. PM2 ile Başlatın

```bash
# Uygulamayı başlatın
pm2 start server.js --name "turkisharmy-backend"

# Sistem başlangıcında otomatik başlatma
pm2 startup
# Çıkan komutu çalıştırın (sudo ile başlayan)

# Mevcut PM2 listesini kaydedin
pm2 save

# Durumu kontrol edin
pm2 status
pm2 logs turkisharmy-backend
```

### 5. PM2 Yönetim Komutları

```bash
# Durumu görüntüle
pm2 status

# Logları görüntüle
pm2 logs turkisharmy-backend

# Yeniden başlat
pm2 restart turkisharmy-backend

# Durdur
pm2 stop turkisharmy-backend

# Sil
pm2 delete turkisharmy-backend

# Monitoring
pm2 monit
```

---

## 🌐 Nginx Yapılandırması

### 1. Nginx Config Dosyası Oluşturun

Backend API için Nginx reverse proxy yapılandırması:

```bash
sudo nano /etc/nginx/sites-available/api.turkisharmycs2.com
```

Aşağıdaki içeriği ekleyin:

```nginx
# Backend API - api.turkisharmycs2.com
server {
    listen 80;
    listen [::]:80;
    server_name api.turkisharmycs2.com;

    # Logs
    access_log /var/log/nginx/api.turkisharmycs2.com.access.log;
    error_log /var/log/nginx/api.turkisharmycs2.com.error.log;

    # Reverse proxy to Node.js backend
    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        
        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        
        # Headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # Cache bypass
        proxy_cache_bypass $http_upgrade;
        
        # Buffer settings
        proxy_buffering off;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/json application/javascript application/xml+rss;
}
```

### 2. Nginx Config'i Aktif Edin

```bash
# Symbolic link oluşturun
sudo ln -s /etc/nginx/sites-available/api.turkisharmycs2.com /etc/nginx/sites-enabled/

# Nginx config'i test edin
sudo nginx -t

# Nginx'i yeniden yükleyin
sudo systemctl reload nginx

# Nginx durumunu kontrol edin
sudo systemctl status nginx
```

### 3. Nginx Yönetim Komutları

```bash
# Nginx'i başlat
sudo systemctl start nginx

# Nginx'i durdur
sudo systemctl stop nginx

# Nginx'i yeniden başlat
sudo systemctl restart nginx

# Nginx'i yeniden yükle (downtime olmadan)
sudo systemctl reload nginx

# Nginx durumunu kontrol et
sudo systemctl status nginx

# Nginx loglarını görüntüle
sudo tail -f /var/log/nginx/api.turkisharmycs2.com.access.log
sudo tail -f /var/log/nginx/api.turkisharmycs2.com.error.log
```

### 4. Firewall Ayarları

```bash
# UFW firewall kullanıyorsanız
sudo ufw allow 'Nginx Full'
sudo ufw allow 22/tcp  # SSH
sudo ufw enable
sudo ufw status
```

---

## 🎨 Frontend Deployment

Frontend'i static hosting servisine deploy edeceğiz (Netlify, Vercel, veya Cloudflare Pages).

### 1. Environment Variables Ayarlayın

Proje dizininde `.env` dosyası oluşturun:

```bash
cd turkisharmy-cs2-website
nano .env
```

İçeriği:

```env
VITE_API_URL=https://api.turkisharmycs2.com
```

### 2. Production Build Oluşturun

```bash
# Dependencies yükleyin
npm install

# Production build
npm run build

# Build klasörü: dist/
```

### 3. Netlify Deployment

#### Option A: Netlify CLI

```bash
# Netlify CLI yükleyin
npm install -g netlify-cli

# Login
netlify login

# Deploy
netlify deploy --prod

# Site klasörü: dist
```

#### Option B: Netlify Web Interface

1. [Netlify](https://app.netlify.com/) hesabınıza giriş yapın
2. "Add new site" → "Import an existing project"
3. GitHub repository'nizi seçin
4. Build settings:
   - **Build command:** `npm run build`
   - **Publish directory:** `dist`
   - **Environment variables:**
     - `VITE_API_URL` = `https://api.turkisharmycs2.com`
5. "Deploy site" butonuna tıklayın

#### Netlify Redirects

`public/_redirects` dosyası zaten mevcut (SPA routing için):

```
/*    /index.html   200
```

### 4. Vercel Deployment

```bash
# Vercel CLI yükleyin
npm install -g vercel

# Login
vercel login

# Deploy
vercel --prod

# Environment variables ekleyin
vercel env add VITE_API_URL production
# Değer: https://api.turkisharmycs2.com
```

### 5. Cloudflare Pages Deployment

1. [Cloudflare Dashboard](https://dash.cloudflare.com/) → Pages
2. "Create a project" → "Connect to Git"
3. Repository'nizi seçin
4. Build settings:
   - **Framework preset:** Vite
   - **Build command:** `npm run build`
   - **Build output directory:** `dist`
   - **Root directory:** `turkisharmy-cs2-website`
5. Environment variables:
   - `VITE_API_URL` = `https://api.turkisharmycs2.com`
6. "Save and Deploy"

---

## 🌍 DNS Ayarları

Cloudflare DNS ayarları:

### 1. Frontend (turkisharmycs2.com)

```
Type: A
Name: @
Content: [Netlify/Vercel/Cloudflare IP]
Proxy: Enabled (Orange cloud)
TTL: Auto
```

```
Type: CNAME
Name: www
Content: turkisharmycs2.com
Proxy: Enabled (Orange cloud)
TTL: Auto
```

### 2. Backend API (api.turkisharmycs2.com)

```
Type: A
Name: api
Content: [Your Linux Server IP]
Proxy: Disabled (Gray cloud) - SSL için gerekli
TTL: Auto
```

**Önemli:** Backend için Cloudflare proxy'sini devre dışı bırakın (Gray cloud), aksi takdirde Let's Encrypt SSL sertifikası alamazsınız.

### 3. DNS Propagation Kontrolü

```bash
# DNS'in yayılmasını kontrol edin
nslookup api.turkisharmycs2.com
nslookup turkisharmycs2.com

# veya
dig api.turkisharmycs2.com
dig turkisharmycs2.com
```

DNS propagation 5-10 dakika sürebilir.

---

## 🔒 SSL Sertifikası

Let's Encrypt ile ücretsiz SSL sertifikası alın.

### 1. Certbot ile SSL Kurulumu

```bash
# SSL sertifikası alın (Nginx otomatik yapılandırma ile)
sudo certbot --nginx -d api.turkisharmycs2.com

# Sertifika yenileme testi
sudo certbot renew --dry-run
```

### 2. Otomatik Yenileme

Certbot otomatik olarak cron job oluşturur. Kontrol edin:

```bash
# Cron job'ı kontrol edin
sudo systemctl status certbot.timer

# Manuel yenileme
sudo certbot renew
```

### 3. SSL Sonrası Nginx Config

Certbot, Nginx config'inizi otomatik olarak güncelleyecek. Sonuç:

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name api.turkisharmycs2.com;

    # SSL certificates
    ssl_certificate /etc/letsencrypt/live/api.turkisharmycs2.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.turkisharmycs2.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # ... rest of config
}

# HTTP to HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name api.turkisharmycs2.com;
    return 301 https://$server_name$request_uri;
}
```

---

## ✅ Test ve Doğrulama

### 1. Backend API Testi

```bash
# Health check
curl https://api.turkisharmycs2.com/

# Servers endpoint
curl https://api.turkisharmycs2.com/api/servers

# User endpoint (session gerektirir)
curl https://api.turkisharmycs2.com/api/user
```

### 2. Frontend Testi

1. Browser'da `https://turkisharmycs2.com` açın
2. Tüm sayfaları test edin:
   - Anasayfa
   - Sunucular
   - Kurallar
   - Kadromuz
   - Discord
   - SkinChanger

### 3. Steam Auth Testi

1. SkinChanger sayfasına gidin
2. "Steam ile Giriş Yap" butonuna tıklayın
3. Steam login sayfasına yönlendirilmelisiniz
4. Login sonrası `https://turkisharmycs2.com/skinchanger?auth=success` adresine dönmelisiniz

### 4. CORS Testi

Browser console'da:

```javascript
fetch('https://api.turkisharmycs2.com/api/servers', {
  credentials: 'include'
})
  .then(r => r.json())
  .then(console.log)
```

### 5. SSL Testi

```bash
# SSL sertifikasını kontrol edin
openssl s_client -connect api.turkisharmycs2.com:443 -servername api.turkisharmycs2.com

# veya online test:
# https://www.ssllabs.com/ssltest/analyze.html?d=api.turkisharmycs2.com
```

---

## 🔧 Troubleshooting

### Backend Sorunları

#### Problem: Backend başlamıyor

```bash
# PM2 loglarını kontrol edin
pm2 logs turkisharmy-backend

# Port kullanımda mı?
sudo lsof -i :3001
sudo netstat -tulpn | grep :3001

# Process'i öldürün
sudo kill -9 [PID]

# Yeniden başlatın
pm2 restart turkisharmy-backend
```

#### Problem: Environment variables okunmuyor

```bash
# .env dosyasını kontrol edin
cat ~/TurkishArmy-Websites/turkisharmy-cs2-backend/.env

# PM2'yi .env ile yeniden başlatın
cd ~/TurkishArmy-Websites/turkisharmy-cs2-backend
pm2 delete turkisharmy-backend
pm2 start server.js --name "turkisharmy-backend"
```

#### Problem: Steam auth çalışmıyor

1. `.env` dosyasında `STEAM_API_KEY` kontrol edin
2. `FRONTEND_URL` ve `BACKEND_URL` doğru mu?
3. Session secret ayarlandı mı?
4. HTTPS kullanılıyor mu? (Production'da zorunlu)

### Nginx Sorunları

#### Problem: 502 Bad Gateway

```bash
# Backend çalışıyor mu?
pm2 status

# Nginx loglarını kontrol edin
sudo tail -f /var/log/nginx/api.turkisharmycs2.com.error.log

# Nginx config'i test edin
sudo nginx -t

# Nginx'i yeniden başlatın
sudo systemctl restart nginx
```

#### Problem: 404 Not Found

```bash
# Nginx config'i kontrol edin
sudo nano /etc/nginx/sites-available/api.turkisharmycs2.com

# Symbolic link var mı?
ls -la /etc/nginx/sites-enabled/

# Nginx'i reload edin
sudo systemctl reload nginx
```

#### Problem: SSL sertifikası alınamıyor

```bash
# DNS doğru mu?
nslookup api.turkisharmycs2.com

# Port 80 açık mı?
sudo ufw status
sudo netstat -tulpn | grep :80

# Cloudflare proxy kapalı mı? (Gray cloud)
# Cloudflare dashboard'dan kontrol edin

# Certbot'u verbose mode'da çalıştırın
sudo certbot --nginx -d api.turkisharmycs2.com --verbose
```

### Frontend Sorunları

#### Problem: API çağrıları başarısız

1. Browser console'da CORS hatası var mı?
2. `VITE_API_URL` environment variable doğru mu?
3. Backend CORS ayarları doğru mu?

```javascript
// Browser console'da test edin
console.log(import.meta.env.VITE_API_URL)
```

#### Problem: Steam auth redirect çalışmıyor

1. Backend `.env` dosyasında `FRONTEND_URL` doğru mu?
2. HTTPS kullanılıyor mu?
3. Session cookies çalışıyor mu?

### CORS Sorunları

#### Problem: CORS hatası alıyorum

Backend `server.js` dosyasında CORS ayarlarını kontrol edin:

```javascript
// Production'da sadece bu domain'lere izin verilmeli:
const allowedOrigins = [
  'https://turkisharmycs2.com',
  'https://www.turkisharmycs2.com'
];
```

### DNS Sorunları

#### Problem: Domain çözümlenmiyor

```bash
# DNS propagation kontrolü
nslookup api.turkisharmycs2.com
dig api.turkisharmycs2.com

# Cloudflare DNS ayarlarını kontrol edin
# A record doğru IP'yi gösteriyor mu?
```

---

## 📊 Monitoring ve Maintenance

### PM2 Monitoring

```bash
# Real-time monitoring
pm2 monit

# CPU ve memory kullanımı
pm2 list

# Detaylı bilgi
pm2 show turkisharmy-backend
```

### Nginx Logs

```bash
# Access logs
sudo tail -f /var/log/nginx/api.turkisharmycs2.com.access.log

# Error logs
sudo tail -f /var/log/nginx/api.turkisharmycs2.com.error.log

# Log rotation (otomatik)
sudo logrotate -f /etc/logrotate.d/nginx
```

### Disk Kullanımı

```bash
# Disk kullanımını kontrol edin
df -h

# PM2 log dosyalarını temizleyin
pm2 flush

# Nginx log dosyalarını temizleyin
sudo truncate -s 0 /var/log/nginx/*.log
```

### Güvenlik Güncellemeleri

```bash
# Sistem güncellemeleri
sudo apt-get update
sudo apt-get upgrade

# Node.js güncellemeleri
sudo npm install -g npm@latest

# PM2 güncellemeleri
sudo npm install -g pm2@latest
pm2 update
```

---

## 🎉 Deployment Tamamlandı!

Artık siteniz production'da çalışıyor! 🚀

### Önemli URL'ler:
- **Frontend:** https://turkisharmycs2.com
- **Backend API:** https://api.turkisharmycs2.com
- **Steam Auth:** https://api.turkisharmycs2.com/auth/steam

### Sonraki Adımlar:
1. ✅ Tüm sayfaları test edin
2. ✅ Steam auth'u test edin
3. ✅ Mobile responsive'i kontrol edin
4. ✅ SEO ayarlarını yapın (SEO-REHBERI.md)
5. ✅ Analytics ekleyin (Google Analytics)
6. ✅ Monitoring kurun (PM2, Nginx logs)
7. ✅ Backup stratejisi oluşturun

### Destek:
Herhangi bir sorun yaşarsanız:
1. Bu rehberdeki Troubleshooting bölümüne bakın
2. PM2 ve Nginx loglarını kontrol edin
3. GitHub Issues'da sorun açın

**Başarılar! 🎮**
