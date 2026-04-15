# 🚀 Setup CakePHP di WSL (Windows)

Panduan setup project CakePHP secara lokal menggunakan WSL (Windows Subsystem for Linux).

---

## 📋 Prerequisites

- Windows 10/11 dengan WSL2 aktif
- Ubuntu di WSL
- Apache2, PHP, MySQL/MariaDB sudah terinstall di WSL
- Git sudah terinstall

---

## ⚙️ Langkah-langkah Setup

### 1. Clone & Pull Project

```bash
git clone https://gitlab.com/username/nama-project.git
cd nama-project
git pull
```

---

### 2. Copy Project ke /var/www/html

```bash
sudo cp -r /path/to/project /var/www/html/
sudo chown -R $USER:$USER /var/www/html/nama-project
```

---

### 3. Buat Folder Cache

```bash
sudo mkdir -p /var/www/html/nama-project/app/tmp/cache/{short,long,forever,models,views}
sudo chmod -R 777 /var/www/html/nama-project/app/tmp/
```

---

### 4. Setup Database

```bash
sudo service mysql start
mysql -u root -e "CREATE DATABASE nama_database;"
mysql -u root nama_database < '/var/www/html/nama-project/file.sql'
```

Pastikan konfigurasi database di `app/Config/database.php` sudah sesuai:

```php
public $default = array(
    'datasource' => 'Database/Mysql',
    'host'       => 'localhost',
    'login'      => 'root',
    'password'   => '',
    'database'   => 'nama_database',
);
```

---

### 5. Buat Virtual Host Apache

```bash
sudo nano /etc/apache2/sites-available/nama.local.conf
```

Isi dengan konfigurasi berikut:

```apache
<VirtualHost *:80>
    ServerName nama.local
    DocumentRoot /var/www/html/nama-project/app/webroot

    <Directory /var/www/html/nama-project/app/webroot>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/nama.local-error.log
    CustomLog ${APACHE_LOG_DIR}/nama.local-access.log combined
</VirtualHost>
```

---

### 6. Enable Vhost & Restart Apache

```bash
sudo a2ensite nama.local.conf
sudo a2dissite 000-default.conf
sudo a2enmod rewrite
sudo service apache2 restart
```

---

### 7. Edit Hosts File Windows

Buka **Notepad as Administrator**, lalu buka file:

```
C:\Windows\System32\drivers\etc\hosts
```

Tambahkan baris berikut di bagian paling bawah:

```
127.0.0.1   nama.local
```

---

### 8. Cek IP WSL

Jalankan perintah berikut di WSL untuk mendapatkan IP:

```bash
hostname -I
```

Catat IP-nya (contoh: `172.25.234.146`).

---

### 9. Setup Port Proxy

Buka **PowerShell as Administrator** di Windows, lalu jalankan:

```powershell
netsh interface portproxy add v4tov4 listenport=80 listenaddress=127.0.0.1 connectport=80 connectaddress=IP_WSL
```

> Ganti `IP_WSL` dengan hasil `hostname -I` tadi.

---

### 10. Flush DNS & Buka Browser

Di **Command Prompt Windows**:

```cmd
ipconfig /flushdns
```

Lalu buka browser dan akses:

```
http://nama.local
```

---

## ⚠️ Catatan Penting

- **IP WSL bisa berubah** setiap kali Windows di-restart. Jika tiba-tiba tidak bisa akses, lakukan langkah berikut:
  1. Cek IP baru: `hostname -I` di WSL
  2. Update portproxy di PowerShell:
     ```powershell
     netsh interface portproxy delete v4tov4 listenport=80 listenaddress=127.0.0.1
     netsh interface portproxy add v4tov4 listenport=80 listenaddress=127.0.0.1 connectport=80 connectaddress=IP_BARU
     ```
  3. Flush DNS: `ipconfig /flushdns`

- Gunakan ekstensi domain **`.local`** atau **`.test`**, hindari **`.app`** karena browser modern otomatis redirect ke HTTPS.

- Pastikan Apache sudah listen di IPv4 dengan menambahkan `Listen 0.0.0.0:80` di `/etc/apache2/ports.conf`.

---

## 🛠️ Troubleshooting

| Error | Solusi |
|-------|--------|
| `ERR_CONNECTION_REFUSED` | Cek portproxy & pastikan Apache jalan (`sudo service apache2 status`) |
| `HTTP ERROR 500` | Cek folder cache sudah dibuat & permission sudah 777 |
| `MissingConnectionException` | Cek konfigurasi database & pastikan MySQL jalan |
| Browser redirect ke HTTPS | Gunakan domain `.local` atau clear HSTS di `chrome://net-internals/#hsts` |
| IP WSL berubah | Update portproxy & hosts file dengan IP baru |