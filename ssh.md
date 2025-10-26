# SteelDev License Platform - Linux Sunucu Kurulum Kılavuzu

Bu kılavuz, platformu Linux sunucuda (Ubuntu/Debian) sıfırdan kurmak için **adım adım** hazırlanmıştır. Hiç bilginiz olmasa bile bu kılavuzu takip ederek kurulum yapabilirsiniz.

## İçindekiler
1. [Discord Uygulaması Oluşturma](#1-discord-uygulaması-oluşturma)
2. [Linux Sunucu Bağlantısı](#2-linux-sunucu-bağlantısı)
3. [Sistem Güncellemeleri](#3-sistem-güncellemeleri)
4. [Node.js Kurulumu](#4-nodejs-kurulumu)
5. [Proje Dosyalarını Yükleme](#5-proje-dosyalarını-yükleme)
6. [Ortam Değişkenlerini Yapılandırma](#6-ortam-değişkenlerini-yapılandırma)
7. [Bağımlılıkları Yükleme ve Build](#7-bağımlılıkları-yükleme-ve-build)
8. [PM2 ile Backend Kurulumu](#8-pm2-ile-backend-kurulumu)
9. [Nginx Web Sunucu Kurulumu](#9-nginx-web-sunucu-kurulumu)
10. [SSL Sertifikası Kurulumu](#10-ssl-sertifikası-kurulumu)
11. [Firewall Ayarları](#11-firewall-ayarları)
12. [Test ve Doğrulama](#12-test-ve-doğrulama)
13. [İlk Giriş ve Admin Oluşturma](#13-i̇lk-giriş-ve-admin-oluşturma)
14. [Otomatik Yedekleme](#14-otomatik-yedekleme)
15. [Sorun Giderme](#15-sorun-giderme)

---

## 1. Discord Uygulaması Oluşturma

Discord OAuth2 ile kullanıcı girişi yapabilmek için bir Discord uygulaması oluşturmanız gerekiyor.

### Adım 1.1: Discord Developer Portal

1. Web tarayıcınızda [Discord Developer Portal](https://discord.com/developers/applications)'a gidin
2. Discord hesabınızla giriş yapın
3. Sağ üst köşedeki **"New Application"** butonuna tıklayın
4. Uygulamanıza bir isim verin (örnek: "SteelDev License System")
5. Şartları kabul edin ve **"Create"** butonuna tıklayın

### Adım 1.2: OAuth2 Ayarları

1. Sol menüden **"OAuth2"** sekmesine tıklayın
2. **"Redirects"** bölümünü bulun
3. **"Add Redirect"** butonuna tıklayın
4. Geliştirme ortamı için şunu ekleyin:
   ```
   http://localhost:5173/auth/callback
   ```
5. Production (canlı) ortamı için şunu ekleyin (domain adınızı yazın):
   ```
   https://yourdomain.com/auth/callback
   ```
6. **"Save Changes"** butonuna tıklayın

### Adım 1.3: Kimlik Bilgilerini Kaydetme

1. Sol menüden **"OAuth2"** > **"General"** sayfasına gidin
2. **Client ID** bilgisini kopyalayın ve bir yere (not defterine) kaydedin
3. **Client Secret** bölümünde **"Reset Secret"** butonuna tıklayın
4. Çıkan secret'ı kopyalayın ve güvenli bir yere kaydedin (bu tekrar gösterilmeyecek!)

> ⚠️ **ÖNEMLİ:** Client Secret'ı kimseyle paylaşmayın ve GitHub'a yüklemeyin!

---

## 2. Linux Sunucu Bağlantısı

Linux sunucunuza SSH ile bağlanmanız gerekiyor.

### Windows Kullanıyorsanız:

**PuTTY ile bağlantı:**
1. [PuTTY'yi indirin](https://www.putty.org/)
2. PuTTY'yi açın
3. **Host Name** alanına sunucu IP adresinizi girin
4. **Port** alanına `22` yazın
5. **"Open"** butonuna tıklayın
6. Kullanıcı adı (genellikle `root` veya `ubuntu`) ve şifrenizi girin

### Mac/Linux Kullanıyorsanız:

Terminal açın ve şu komutu yazın:
```bash
ssh root@SUNUCU_IP_ADRESI
```
(SUNUCU_IP_ADRESI yerine kendi IP adresinizi yazın)

Şifrenizi girin ve Enter tuşuna basın.

---

## 3. Sistem Güncellemeleri

Sunucuya bağlandıktan sonra ilk olarak sistemi güncellememiz gerekiyor.

```bash
# Paket listesini güncelle
sudo apt update

# Kurulu paketleri güncelle
sudo apt upgrade -y

# Gerekli temel araçları kur
sudo apt install -y curl wget git nano ufw
```

**Ne yaptık?**
- `apt update` → Yeni paket listesini indirdi
- `apt upgrade -y` → Sistemdeki tüm paketleri güncelledi
- `apt install` → curl, wget, git, nano, ufw gibi temel araçları yükledi

> 💡 **Bilgi:** `-y` parametresi "evet" demek anlamına gelir, böylece onay istemeden kurulum yapar.

---

## 4. Node.js Kurulumu

Projemiz Node.js ile çalıştığı için Node.js'i kurmamız gerekiyor.

### Adım 4.1: NodeSource Deposunu Ekleyin

```bash
# NodeSource deposunu ekle (Node.js 20.x LTS)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

**Ne yaptık?**
- NodeSource'un resmi deposunu sisteme ekledik
- Bu sayede en güncel Node.js 20.x LTS sürümünü kurabileceğiz

### Adım 4.2: Node.js'i Kurun

```bash
# Node.js ve npm'i kur
sudo apt install -y nodejs

# Kurulumu kontrol et
node --version
npm --version
```

**Beklenen çıktı:**
```
v20.x.x
10.x.x
```

> ✅ Bu sürüm numaralarını görüyorsanız Node.js başarıyla kurulmuştur!

---

## 5. Proje Dosyalarını Yükleme

Proje dosyalarını sunucuya yüklemenin 3 farklı yolu var:

### Yöntem 1: Git ile Clone (ÖNERİLEN)

Projeniz GitHub/GitLab'da ise:

```bash
# Proje klasörüne git
cd /var/www

# Git repository'sini clone et
git clone https://github.com/KULLANICI_ADI/REPO_ADI.git steeldev-license

# Proje klasörüne gir
cd steeldev-license
```

### Yöntem 2: FTP/SFTP ile Yükleme

FileZilla veya WinSCP kullanarak:

1. **FileZilla**'yı açın
2. Sunucu bilgilerini girin:
   - Host: `sftp://SUNUCU_IP`
   - Username: `root`
   - Password: `şifreniz`
   - Port: `22`
3. Bağlan'a tıklayın
4. Proje dosyalarını `/var/www/steeldev-license` klasörüne yükleyin

### Yöntem 3: SCP ile Yükleme (Bilgisayarınızdan)

Kendi bilgisayarınızdan (Mac/Linux terminal):

```bash
# Proje klasörünüzün içindeyken
scp -r ./ root@SUNUCU_IP:/var/www/steeldev-license
```

### Klasör İzinlerini Ayarlayın

```bash
# steeldev-license klasörüne git
cd /var/www/steeldev-license

# Dosya sahipliğini ayarla
sudo chown -R $USER:$USER /var/www/steeldev-license

# İzinleri ayarla
sudo chmod -R 755 /var/www/steeldev-license
```

---

## 6. Ortam Değişkenlerini Yapılandırma

Uygulamamızın çalışması için gerekli ayarları yapacağız.

### Adım 6.1: Frontend .env Dosyası

Proje ana dizininde `.env` dosyasını düzenleyin:

```bash
cd /var/www/steeldev-license
nano .env
```

**Dosya içeriği:**

```env
# Supabase Configuration (ZATEN AYARLI - DEĞİŞTİRMEYİN)
VITE_SUPABASE_URL=https://mcornmtovhbmugwzcipu.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im1jb3JubXRvdmhibXVnd3pjaXB1Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3Mjk5NjY2NzEsImV4cCI6MjA0NTU0MjY3MX0.5Fq8cqPjVx6cSBZDLlzM_qS1OhVMGHsEXwgQqBAVGrg

# Discord OAuth2 Configuration (BURAYA KENDİ BİLGİLERİNİZİ GİRİN)
VITE_DISCORD_CLIENT_ID=your_discord_client_id_buraya
VITE_DISCORD_CLIENT_SECRET=your_discord_client_secret_buraya
VITE_DISCORD_REDIRECT_URI=http://localhost:5173/auth/callback

# Backend License Server (Şimdilik localhost, sonra değiştirecez)
VITE_LICENSE_SERVER_URL=http://localhost:3001
```

**Kaydetme:**
- `CTRL + O` → Kaydet
- `ENTER` → Onayla
- `CTRL + X` → Çık

### Adım 6.2: Backend .env Dosyası Oluşturma

Backend için ayrı bir `.env` dosyası oluşturalım:

```bash
nano server/.env
```

**Dosya içeriği:**

```env
# JWT Secrets (Aşağıdaki komutu çalıştırıp buraya yapıştırın)
LICENSE_JWT_SECRET=buraya_random_secret_gelecek
ADMIN_JWT_SECRET=buraya_random_secret_gelecek

# Server Configuration
PORT=3001
ALLOWED_ORIGIN=http://localhost:5173

# Supabase (Opsiyonel - Backend için)
SUPABASE_URL=https://mcornmtovhbmugwzcipu.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key_buraya_gelecek
```

### Adım 6.3: JWT Secret Oluşturma

Güvenli random secret'lar oluşturalım:

```bash
# İlk secret (LICENSE_JWT_SECRET)
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"

# İkinci secret (ADMIN_JWT_SECRET)
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

**Çıkan sonuçları kopyalayın ve `server/.env` dosyasındaki ilgili yerlere yapıştırın.**

Örnek çıktı:
```
XtR9kL2mP4vN8qY6wJ3sH7bG5nK0aZ1dF4cM9xE2yU=
```

---

## 7. Bağımlılıkları Yükleme ve Build

Şimdi projenin çalışması için gerekli kütüphaneleri kuracağız.

```bash
# Proje klasörüne git
cd /var/www/steeldev-license

# Node modüllerini kur (bu 1-2 dakika sürebilir)
npm install

# Production build yap
npm run build
```

**Ne yaptık?**
- `npm install` → package.json'daki tüm bağımlılıkları yükledi
- `npm run build` → React uygulamasını production için derleyip `dist/` klasörüne koydu

**Beklenen çıktı:**
```
✓ 1563 modules transformed.
✓ built in 4.66s
```

> ✅ Hata almadan "built in X.XXs" mesajını görüyorsanız başarılıdır!

---

## 8. PM2 ile Backend Kurulumu

PM2, Node.js uygulamalarını arka planda sürekli çalıştıran bir araçtır.

### Adım 8.1: PM2 Kurulumu

```bash
# PM2'yi global olarak kur
sudo npm install -g pm2

# Kurulumu kontrol et
pm2 --version
```

### Adım 8.2: Backend'i PM2 ile Başlatma

```bash
# Proje klasörüne git
cd /var/www/steeldev-license

# Backend'i PM2 ile başlat
pm2 start server/licenseServer.js --name steeldev-backend

# PM2'yi sistem başlangıcında otomatik başlat
pm2 startup systemd
```

**Bu komut size bir komut verecek, o komutu çalıştırın.**

Örnek:
```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u root --hp /root
```

Sonra:

```bash
# Mevcut PM2 süreçlerini kaydet
pm2 save
```

### Adım 8.3: PM2 Durumunu Kontrol Etme

```bash
# Çalışan süreçleri listele
pm2 list

# Detaylı bilgi
pm2 info steeldev-backend

# Canlı logları izle
pm2 logs steeldev-backend

# Logları durdurmak için CTRL+C
```

**PM2 Temel Komutları:**

```bash
pm2 start steeldev-backend    # Başlat
pm2 stop steeldev-backend     # Durdur
pm2 restart steeldev-backend  # Yeniden başlat
pm2 delete steeldev-backend   # Sil
pm2 logs steeldev-backend     # Logları göster
pm2 monit                     # Canlı izleme ekranı
```

---

## 9. Nginx Web Sunucu Kurulumu

Nginx, web sayfalarını sunmak ve backend'e proxy yapmak için kullanılır.

### Nginx Nedir?

**Basit anlatım:**
- Nginx bir web sunucusudur
- Kullanıcı tarayıcıdan siteyi açar → Nginx karşılar
- Nginx static dosyaları (HTML, CSS, JS) direkt sunar
- API istekleri gelirse → Backend'e (PM2'deki uygulama) yönlendirir

```
[Kullanıcı Tarayıcısı]
        ↓ (HTTP/HTTPS)
    [Nginx :80/:443]
        ↓
    ┌─────────┬─────────┐
    ↓         ↓         ↓
[Static]  [/api]  [/health]
[Files]     ↓         ↓
(dist/)  [Backend :3001]
```

### Adım 9.1: Nginx Kurulumu

```bash
# Nginx'i kur
sudo apt install -y nginx

# Kurulumu kontrol et
nginx -v

# Nginx'i başlat
sudo systemctl start nginx

# Sistem açılışında otomatik başlat
sudo systemctl enable nginx

# Durumu kontrol et
sudo systemctl status nginx
```

**Beklenen çıktı:**
```
● nginx.service - A high performance web server
   Active: active (running)
```

> ✅ "active (running)" görüyorsanız Nginx çalışıyor demektir!

**Tarayıcıda test edin:**
```
http://SUNUCU_IP_ADRESI
```
Nginx'in varsayılan "Welcome to nginx!" sayfasını görmelisiniz.

### Adım 9.2: Nginx Yapılandırma Dosyası Oluşturma

Şimdi projemiz için özel bir Nginx ayar dosyası oluşturacağız.

```bash
# Yeni site config dosyası oluştur
sudo nano /etc/nginx/sites-available/steeldev-license
```

**Dosya içeriği:** (Tamamını kopyalayıp yapıştırın)

```nginx
# Frontend Server (React App)
server {
    listen 80;
    listen [::]:80;

    # Domain adınızı buraya yazın (yoksa sadece IP kullanabilirsiniz)
    server_name yourdomain.com www.yourdomain.com;

    # Root dizin (React build dosyaları)
    root /var/www/steeldev-license/dist;
    index index.html;

    # Gzip compression (dosya boyutlarını küçültür)
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript
               application/x-javascript application/xml+rss
               application/javascript application/json;

    # Ana sayfa - React Router için
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Backend API Proxy
    location /api {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;

        # WebSocket desteği
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';

        # Diğer header'lar
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeout ayarları
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Health check endpoint
    location /health {
        proxy_pass http://localhost:3001;
        proxy_set_header Host $host;
    }

    # Static dosyalar için cache
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;

    # Güvenlik - hassas dosyaları gizle
    location ~ /\. {
        deny all;
    }

    location ~ /\.env {
        deny all;
    }
}
```

**Ne yaptık? - Satır satır açıklama:**

| Satır | Açıklama |
|-------|----------|
| `listen 80;` | HTTP portunda (80) dinle |
| `server_name yourdomain.com;` | Domain adı (şimdilik bunu kendinize göre değiştirin) |
| `root /var/www/...;` | React build dosyalarının olduğu klasör |
| `location / { ... }` | Ana sayfa istekleri buraya gelir |
| `try_files $uri $uri/ /index.html;` | React Router için - her istekte index.html'i döndür |
| `location /api { ... }` | `/api` ile başlayan istekler backend'e yönlendir |
| `proxy_pass http://localhost:3001;` | Backend'in adresi (PM2'deki uygulama) |
| `gzip on;` | Dosyaları sıkıştır (hız artar) |
| `add_header ...` | Güvenlik başlıkları ekle |

**Kaydetme:**
- `CTRL + O` → Kaydet
- `ENTER` → Onayla
- `CTRL + X` → Çık

### Adım 9.3: Site Yapılandırmasını Aktifleştirme

```bash
# Symbolic link oluştur (dosyayı aktifleştir)
sudo ln -s /etc/nginx/sites-available/steeldev-license /etc/nginx/sites-enabled/

# Varsayılan Nginx sayfasını kaldır (opsiyonel)
sudo rm /etc/nginx/sites-enabled/default

# Nginx config dosyasını test et (hata var mı?)
sudo nginx -t
```

**Beklenen çıktı:**
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

> ✅ "test is successful" görüyorsanız config doğru demektir!

### Adım 9.4: Nginx'i Yeniden Başlatma

```bash
# Nginx'i yeniden yükle (yeni config'i uygula)
sudo systemctl reload nginx

# Veya yeniden başlat
sudo systemctl restart nginx

# Durumu kontrol et
sudo systemctl status nginx
```

**Nginx Temel Komutları:**

```bash
sudo systemctl start nginx      # Başlat
sudo systemctl stop nginx       # Durdur
sudo systemctl restart nginx    # Yeniden başlat
sudo systemctl reload nginx     # Config'i yeniden yükle (kesinti olmadan)
sudo systemctl status nginx     # Durum kontrol
sudo nginx -t                   # Config test
```

### Adım 9.5: Nginx Log Dosyaları

Hata olursa logları kontrol edin:

```bash
# Access log (gelen tüm istekler)
sudo tail -f /var/log/nginx/access.log

# Error log (hatalar)
sudo tail -f /var/log/nginx/error.log

# Son 50 satırı göster
sudo tail -n 50 /var/log/nginx/error.log
```

---

## 10. SSL Sertifikası Kurulumu (Let's Encrypt)

SSL sertifikası, sitenizin HTTPS ile güvenli çalışmasını sağlar.

### SSL Nedir?

**Basit anlatım:**
- SSL olmadan: `http://site.com` → Güvensiz, şifrelenmemiş
- SSL ile: `https://site.com` → Güvenli, şifrelenmiş, tarayıcıda kilit simgesi

**Let's Encrypt:** Ücretsiz SSL sertifikası sağlayan bir servistir.

### Adım 10.1: Certbot Kurulumu

```bash
# Certbot ve Nginx eklentisini kur
sudo apt install -y certbot python3-certbot-nginx

# Kurulumu kontrol et
certbot --version
```

### Adım 10.2: SSL Sertifikası Alma

**ÖNEMLİ HAZIRLIK:**
- Domain adınızın A kaydı sunucu IP'nize yönlendirilmiş olmalı
- Domain adı 80 portuna erişebilmeli (firewall açık olmalı)

```bash
# SSL sertifikası al ve otomatik Nginx config'i güncelle
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

**Certbot size şunları soracak:**

1. **Email adresi:** Yenileme bildirimleri için (gerçek email girin)
2. **Terms of Service:** `Y` yazın (kabul et)
3. **Share email:** `N` yazabilirsiniz (opsiyonel)
4. **Redirect HTTP to HTTPS:** `2` seçin (her zaman HTTPS kullan)

**Beklenen çıktı:**
```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/yourdomain.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/yourdomain.com/privkey.pem
```

> ✅ "Successfully received certificate" görürseniz SSL kurulmuştur!

### Adım 10.3: Otomatik Yenileme Testi

Let's Encrypt sertifikaları 90 günde bir yenilenir. Otomatik yenilemeyi test edelim:

```bash
# Dry run (gerçek yenileme yapmadan test)
sudo certbot renew --dry-run
```

**Beklenen çıktı:**
```
Congratulations, all simulated renewals succeeded
```

> ✅ Bu mesajı görüyorsanız otomatik yenileme çalışacaktır!

### Adım 10.4: SSL Sonrası Nginx Config

Certbot otomatik olarak Nginx config'inizi günceller. Kontrol edelim:

```bash
# Config dosyasını görüntüle
sudo nano /etc/nginx/sites-available/steeldev-license
```

Şimdi dosyada şunlar olmalı:
- `listen 443 ssl;` → HTTPS portu
- `ssl_certificate` → Sertifika yolu
- `ssl_certificate_key` → Private key yolu
- HTTP'den HTTPS'e yönlendirme

**Manuel SSL Ekleme (Certbot başarısız olursa):**

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL Sertifikaları
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/yourdomain.com/chain.pem;

    # SSL Ayarları
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # HSTS (Strict Transport Security)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # ... (geri kalan location blokları aynı)
}

# HTTP'den HTTPS'e yönlendirme
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$server_name$request_uri;
}
```

Nginx'i yeniden yükleyin:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 11. Firewall Ayarları

UFW (Uncomplicated Firewall) ile güvenlik duvarını yapılandıracağız.

### Firewall Nedir?

**Basit anlatım:**
- Firewall, sunucunuza hangi portlardan erişilebileceğini kontrol eder
- Sadece gerekli portları açarsınız, geri kalanı kapalıdır
- Bu sayede güvenlik artar

### Adım 11.1: UFW Kurulumu ve Ayarları

```bash
# UFW kurulu mu kontrol et
sudo ufw status

# SSH portunu aç (ÖNEMLİ: Bunu yapmazsanız bağlantınız kopar!)
sudo ufw allow 22/tcp

# HTTP (Nginx)
sudo ufw allow 80/tcp

# HTTPS (Nginx SSL)
sudo ufw allow 443/tcp

# UFW'yi aktifleştir
sudo ufw enable

# Durumu kontrol et
sudo ufw status verbose
```

**Beklenen çıktı:**
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

### Adım 11.2: UFW Temel Komutları

```bash
sudo ufw status              # Durum göster
sudo ufw enable              # Aktifleştir
sudo ufw disable             # Devre dışı bırak
sudo ufw allow 80/tcp        # Port aç
sudo ufw deny 80/tcp         # Port kapat
sudo ufw delete allow 80/tcp # Kuralı sil
sudo ufw reset               # Tüm kuralları sıfırla
```

---

## 12. Test ve Doğrulama

Her şey kuruldu! Şimdi test edelim.

### Adım 12.1: Backend Testi

```bash
# Backend çalışıyor mu?
curl http://localhost:3001/health

# PM2 durumu
pm2 list

# PM2 logları
pm2 logs steeldev-backend --lines 50
```

**Beklenen çıktı:**
```json
{"status":"ok","message":"License server is running"}
```

### Adım 12.2: Nginx Testi

```bash
# Nginx durumu
sudo systemctl status nginx

# Config testi
sudo nginx -t

# Port dinleme kontrolü
sudo netstat -tulpn | grep nginx
```

**Beklenen çıktı:**
```
tcp  0  0  0.0.0.0:80     0.0.0.0:*  LISTEN  1234/nginx
tcp  0  0  0.0.0.0:443    0.0.0.0:*  LISTEN  1234/nginx
```

### Adım 12.3: Frontend Testi (Tarayıcıda)

Tarayıcınızda şu adresleri açın:

```
https://yourdomain.com
```

veya sadece IP ile:

```
http://SUNUCU_IP_ADRESI
```

**Görmemiz gerekenler:**
- ✅ SteelDev License ana sayfası
- ✅ "Discord ile Giriş Yap" butonu
- ✅ Tarayıcıda kilit simgesi (HTTPS ise)

### Adım 12.4: API Proxy Testi

```bash
# Health endpoint testi
curl https://yourdomain.com/health

# Veya IP ile
curl http://SUNUCU_IP/health
```

**Beklenen çıktı:**
```json
{"status":"ok","message":"License server is running"}
```

### Adım 12.5: Log Kontrolü

Hata varsa logları kontrol edin:

```bash
# PM2 backend logs
pm2 logs steeldev-backend

# Nginx access log
sudo tail -f /var/log/nginx/access.log

# Nginx error log
sudo tail -f /var/log/nginx/error.log

# Sistem logs
sudo journalctl -u nginx -n 50
```

---

## 13. İlk Giriş ve Admin Oluşturma

### Adım 13.1: Production Ortam Değişkenlerini Güncelleme

SSL kurduktan sonra `.env` dosyasını güncellememiz gerekiyor:

```bash
cd /var/www/steeldev-license
nano .env
```

**Güncellenecek satırlar:**

```env
# Discord Redirect URI'yi HTTPS yap
VITE_DISCORD_REDIRECT_URI=https://yourdomain.com/auth/callback

# License Server URL'yi HTTPS yap
VITE_LICENSE_SERVER_URL=https://yourdomain.com

# CORS için allowed origin
ALLOWED_ORIGIN=https://yourdomain.com
```

**Backend .env dosyasını da güncelleyin:**

```bash
nano server/.env
```

```env
ALLOWED_ORIGIN=https://yourdomain.com
```

**Yeniden build yapın:**

```bash
npm run build

# Backend'i yeniden başlat
pm2 restart steeldev-backend
```

### Adım 13.2: Discord OAuth Redirect URI Güncelleme

1. [Discord Developer Portal](https://discord.com/developers/applications)'a gidin
2. Uygulamanızı seçin
3. **OAuth2** > **Redirects** bölümüne gidin
4. Yeni redirect URL ekleyin:
   ```
   https://yourdomain.com/auth/callback
   ```
5. **Save Changes** tıklayın

### Adım 13.3: İlk Giriş

1. Tarayıcıda `https://yourdomain.com` açın
2. **"Discord ile Giriş Yap"** butonuna tıklayın
3. Discord yetkilendirme sayfası açılacak
4. **"Yetkilendir"** butonuna tıklayın
5. Dashboard'a yönlendirileceksiniz

### Adım 13.4: Admin Yetkisi Verme

İlk kullanıcıya admin yetkisi vermek için Supabase'de SQL çalıştırmamız gerekiyor.

**Discord ID'nizi öğrenme:**
1. Discord'u açın
2. Ayarlar > Gelişmiş > Geliştirici Modu'nu açın
3. Profilinize sağ tıklayın
4. **"Kullanıcı Kimliğini Kopyala"** seçin

**Supabase SQL Editor:**

1. [Supabase Dashboard](https://supabase.com/dashboard) açın
2. **SQL Editor** sekmesine gidin
3. **New Query** tıklayın
4. Şu SQL'i çalıştırın:

```sql
-- Discord ID'nizi buraya yazın
UPDATE users
SET role = 'admin'
WHERE discord_id = 'YOUR_DISCORD_ID_BURAYA';

-- Kontrol edin
SELECT discord_username, discord_id, email, role
FROM users
WHERE role = 'admin';
```

5. **RUN** butonuna tıklayın

> ✅ Query başarılı ise artık admin yetkileriniz var!

### Adım 13.5: Admin Panel Erişimi

1. Sayfayı yenileyin (F5)
2. Navbar'da **"Admin Panel"** butonu görünmeli
3. Admin Panel'e tıklayın
4. Ürün ekleme, kullanıcı yönetimi ve lisans yönetimi yapabilirsiniz

---

## 14. Otomatik Yedekleme

Düzenli yedek almak çok önemlidir!

### Adım 14.1: Yedekleme Script Oluşturma

```bash
# Scripts klasörü oluştur
sudo mkdir -p /opt/backups

# Backup script oluştur
sudo nano /opt/backups/backup-steeldev.sh
```

**Script içeriği:**

```bash
#!/bin/bash

# Tarih damgası
DATE=$(date +%Y-%m-%d-%H%M%S)

# Yedek klasörü
BACKUP_DIR="/opt/backups"
PROJECT_DIR="/var/www/steeldev-license"

# Yedek dosya adı
BACKUP_FILE="$BACKUP_DIR/steeldev-backup-$DATE.tar.gz"

# Proje klasörünü yedekle (node_modules hariç)
echo "Yedekleme başlıyor: $BACKUP_FILE"
tar -czf $BACKUP_FILE \
    --exclude='node_modules' \
    --exclude='dist' \
    -C /var/www steeldev-license

# Eski yedekleri sil (30 günden eski)
find $BACKUP_DIR -name "steeldev-backup-*.tar.gz" -mtime +30 -delete

echo "Yedekleme tamamlandı: $BACKUP_FILE"
echo "Dosya boyutu: $(du -h $BACKUP_FILE | cut -f1)"
```

**Script'i çalıştırılabilir yap:**

```bash
sudo chmod +x /opt/backups/backup-steeldev.sh
```

**Manuel test:**

```bash
sudo /opt/backups/backup-steeldev.sh
```

### Adım 14.2: Otomatik Yedekleme (Cron Job)

Cron, Linux'ta zamanlanmış görevler için kullanılır.

```bash
# Crontab düzenle
sudo crontab -e
```

**İlk açılışta editör seçimi:** `nano` için `1` seçin ve Enter.

**Dosyanın sonuna ekleyin:**

```bash
# Her gün saat 03:00'da yedek al
0 3 * * * /opt/backups/backup-steeldev.sh >> /var/log/steeldev-backup.log 2>&1

# Her hafta Pazar günü 04:00'da (opsiyonel)
0 4 * * 0 /opt/backups/backup-steeldev.sh >> /var/log/steeldev-backup.log 2>&1
```

**Cron zamanları:**

```
* * * * *
│ │ │ │ │
│ │ │ │ └─ Haftanın günü (0-7, 0=Pazar)
│ │ │ └─── Ay (1-12)
│ │ └───── Ayın günü (1-31)
│ └─────── Saat (0-23)
└───────── Dakika (0-59)
```

**Örnekler:**

```bash
0 3 * * *      # Her gün 03:00'da
0 */6 * * *    # Her 6 saatte bir
0 0 * * 0      # Her Pazar 00:00'da
*/30 * * * *   # Her 30 dakikada bir
```

**Kaydetme:**
- `CTRL + O` → Kaydet
- `ENTER` → Onayla
- `CTRL + X` → Çık

**Cron job'ları listele:**

```bash
sudo crontab -l
```

### Adım 14.3: Yedeği Geri Yükleme

```bash
# Yedekleri listele
ls -lh /opt/backups/

# Yedeği geri yükle
cd /var/www
sudo tar -xzf /opt/backups/steeldev-backup-2024-01-15-030000.tar.gz

# Bağımlılıkları yeniden yükle
cd steeldev-license
npm install
npm run build

# Servisleri yeniden başlat
pm2 restart steeldev-backend
sudo systemctl reload nginx
```

---

## 15. Sorun Giderme

### Sorun 1: Frontend Sayfası Açılmıyor

**Kontrol adımları:**

```bash
# 1. Nginx çalışıyor mu?
sudo systemctl status nginx

# 2. Build dosyaları var mı?
ls -la /var/www/steeldev-license/dist/

# 3. Nginx config doğru mu?
sudo nginx -t

# 4. Nginx error log
sudo tail -n 50 /var/log/nginx/error.log

# Çözüm: Nginx'i yeniden başlat
sudo systemctl restart nginx
```

### Sorun 2: Backend Bağlantı Hatası (API çalışmıyor)

**Kontrol adımları:**

```bash
# 1. PM2 çalışıyor mu?
pm2 list

# 2. Backend logları
pm2 logs steeldev-backend --lines 100

# 3. Backend'e direkt erişim
curl http://localhost:3001/health

# 4. Port dinleme kontrolü
sudo netstat -tulpn | grep 3001

# Çözüm: Backend'i yeniden başlat
pm2 restart steeldev-backend

# Veya durdur ve başlat
pm2 stop steeldev-backend
pm2 start server/licenseServer.js --name steeldev-backend
pm2 save
```

### Sorun 3: Discord Auth Çalışmıyor

**Kontrol listesi:**

```bash
# 1. .env dosyasını kontrol et
cat .env | grep DISCORD

# 2. Discord Developer Portal kontrol:
# - Client ID doğru mu?
# - Client Secret doğru mu?
# - Redirect URI eklenmiş mi?
#   https://yourdomain.com/auth/callback

# 3. Browser console'da hata var mı?
# Tarayıcıda F12 > Console
```

**Çözüm:**
1. Discord Developer Portal'da Redirect URI'yi kontrol edin
2. `.env` dosyasındaki bilgileri kontrol edin
3. `npm run build` yapıp Nginx'i yeniden yükleyin

```bash
cd /var/www/steeldev-license
npm run build
sudo systemctl reload nginx
```

### Sorun 4: "502 Bad Gateway" Hatası

**Anlamı:** Nginx backend'e ulaşamıyor.

**Çözüm adımları:**

```bash
# 1. Backend çalışıyor mu?
pm2 list

# 2. Backend'i başlat
pm2 restart steeldev-backend

# 3. Port kontrolü
sudo netstat -tulpn | grep 3001

# 4. Firewall backend portunu kapıyor mu?
sudo ufw status

# 5. SELinux sorunu olabilir (CentOS/RHEL)
sudo setenforce 0
```

### Sorun 5: SSL Sertifikası Yüklenmiyor

**Kontrol adımları:**

```bash
# 1. Domain DNS'i doğru mu?
nslookup yourdomain.com

# 2. Port 80 açık mı?
sudo ufw status | grep 80

# 3. Nginx çalışıyor mu?
sudo systemctl status nginx

# 4. Certbot logları
sudo cat /var/log/letsencrypt/letsencrypt.log

# Çözüm: Manuel SSL kurulumu
sudo certbot certonly --standalone -d yourdomain.com
```

**SSL yenileme hatası:**

```bash
# Test et
sudo certbot renew --dry-run

# Force renewal
sudo certbot renew --force-renewal
```

### Sorun 6: "Permission Denied" Hataları

```bash
# Dosya sahipliğini düzelt
sudo chown -R $USER:$USER /var/www/steeldev-license

# İzinleri düzelt
sudo chmod -R 755 /var/www/steeldev-license

# Nginx kullanıcısını kontrol et
ps aux | grep nginx

# Nginx dosyalara erişebiliyor mu?
sudo -u www-data ls /var/www/steeldev-license/dist
```

### Sorun 7: PM2 Sistem Başlangıcında Çalışmıyor

```bash
# PM2 startup'ı yeniden yapılandır
pm2 unstartup
pm2 startup systemd

# Çıkan komutu çalıştırın

# Mevcut uygulamaları kaydet
pm2 save

# Sistemi yeniden başlat ve test et
sudo reboot
```

### Faydalı Komutlar

```bash
# Sistem kaynakları
htop                              # Canlı sistem izleme (apt install htop)
free -h                           # Memory kullanımı
df -h                             # Disk kullanımı
uptime                            # Sistem uptime

# Network
sudo netstat -tulpn               # Açık portlar
sudo ss -tulpn                    # Açık portlar (modern)
curl ifconfig.me                  # Sunucu IP'si
ping google.com                   # İnternet bağlantısı

# Servisler
sudo systemctl status nginx       # Nginx durumu
sudo systemctl status ssh         # SSH durumu
pm2 list                          # PM2 süreçleri

# Loglar
sudo journalctl -xe               # Sistem logları
sudo tail -f /var/log/syslog      # Canlı sistem logu
pm2 logs --lines 100              # PM2 logları

# Dosya işlemleri
ls -lah                           # Detaylı dosya listesi
du -sh *                          # Klasör boyutları
find . -name "*.log"              # Log dosyalarını bul
```

---

## Ek Bilgiler

### Güvenlik İpuçları

1. **SSH Şifresiz Giriş (SSH Key):**
```bash
# Kendi bilgisayarınızda
ssh-keygen -t ed25519

# Public key'i sunucuya kopyala
ssh-copy-id root@SUNUCU_IP
```

2. **SSH Port Değiştirme:**
```bash
sudo nano /etc/ssh/sshd_config
# Port 22 → Port 2222 yap
sudo systemctl restart ssh
sudo ufw allow 2222/tcp
```

3. **Root Girişini Kapat:**
```bash
# Önce yeni kullanıcı oluştur
sudo adduser deploy
sudo usermod -aG sudo deploy

# Root SSH'ı kapat
sudo nano /etc/ssh/sshd_config
# PermitRootLogin yes → no
sudo systemctl restart ssh
```

4. **Fail2Ban Kurulumu (Brute force koruması):**
```bash
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Performans İyileştirmeleri

1. **Nginx Worker Processes:**
```bash
sudo nano /etc/nginx/nginx.conf

# CPU çekirdek sayısı kadar worker
worker_processes auto;
worker_connections 2048;
```

2. **PM2 Cluster Mode:**
```bash
pm2 delete steeldev-backend
pm2 start server/licenseServer.js -i max --name steeldev-backend
pm2 save
```

### Güncelleme Prosedürü

Yeni bir versiyon deploy ederken:

```bash
# 1. Yedek al
sudo /opt/backups/backup-steeldev.sh

# 2. Projeye git
cd /var/www/steeldev-license

# 3. Git pull veya dosyaları güncelle
git pull origin main

# 4. Dependencies güncelle
npm install

# 5. Build yap
npm run build

# 6. Backend'i yeniden başlat
pm2 restart steeldev-backend

# 7. Nginx'i reload et
sudo systemctl reload nginx

# 8. Test et
curl https://yourdomain.com/health
```

### Hızlı Referans Kartı

**Servis Yönetimi:**
```bash
# Nginx
sudo systemctl {start|stop|restart|reload|status} nginx

# PM2
pm2 {start|stop|restart|list|logs|monit} steeldev-backend
```

**Dosya Konumları:**
```
Proje:           /var/www/steeldev-license
Nginx Config:    /etc/nginx/sites-available/steeldev-license
SSL Sertifika:   /etc/letsencrypt/live/yourdomain.com/
Nginx Logs:      /var/log/nginx/
Yedekler:        /opt/backups/
```

**Önemli Portlar:**
```
SSH:        22
HTTP:       80
HTTPS:      443
Backend:    3001 (sadece localhost)
```

---

🎉 **Kurulum Tamamlandı!** Artık SteelDev License Platform Linux sunucuda canlıda çalışıyor.

Sorularınız için: support@steeldev.com
