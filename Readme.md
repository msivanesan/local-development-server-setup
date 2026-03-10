# 📘 COMPLETE DOCUMENTATION

## LAN DEVELOPMENT SERVER (PHP + Flask + Django + Storage)

---

# 🔑 CREDENTIALS OVERVIEW

| Service / User Type | Username | Password | Details |
| :--- | :--- | :--- | :--- |
| **System Admin (Root)** | `devadmin` | `Dev@123` | Created during installation |
| **Desktop / RDP User** | `devuser` | `Dev@123` | For GUI Access (Step 6) |
| **Samba (Storage)** | `devadmin` | `Dev@123` | For File Sharing |
| **MySQL User** | `devadmin` | `Dev@123` | For Database Access |

> **⚠️ IMPORTANT:** These are credentials for real environment.

---

---

# 🧱 OVERVIEW (FINAL RESULT)

After completing this guide:

* One PC runs **Ubuntu**
* Acts as a **development server**
* Accessible from **any Windows PC on LAN**
* Supports **multiple projects**
* Files edited directly from **Windows**
* URLs look like:

```
http://192.168.5.25/project1
http://192.168.5.25/convex_msql
http://192.168.5.25/shop
```

* Storage mapped as a **Windows drive**:

```
Z:\php
Z:\flask
Z:\django
```

![Image](https://media-zep-analytics.s3.ap-south-1.amazonaws.com/uploads/blogs/1685528882.png)

![Image](https://www.conceptdraw.com/How-To-Guide/picture/Computer-and-networks-Local-area-network-diagram.png)

![Image](https://www.cs.iastate.edu/files/inline-images/windowsMap.PNG)

---

# 🔹 STEP 1 — DOWNLOAD OPERATING SYSTEM

## Download

Download **Ubuntu 22.04 Live Server LTS**
(Server is more lightweight and efficient)

---

# 🔹 STEP 2 — CREATE BOOTABLE USB (WINDOWS)

1. Download **Rufus**
2. Insert USB (8 GB+)
3. Rufus settings:

   * Partition scheme: **GPT**
   * Target system: **UEFI**
   * File system: **FAT32**
4. Select Ubuntu ISO
5. Click **START**

---

# 🔹 STEP 3 — INSTALL UBUNTU SERVER

1. Boot PC from USB
2. Select **Try or Install Ubuntu Server**
3. Language/Keyboard: **English**
4. Network: Keep default (DHCP) for now
5. Storage:

   ```
   Use an entire disk
   (Uncheck LVM if not needed, or keep default)
   ```
6. Profile Setup:

   ```
   Your name: Dev Admin
   Your server's name: dev-server
   Pick a username: devadmin
   Choose a password: Dev@123
   ```
7. SSH Setup:

   * [x] **Install OpenSSH server** (Important)
8. Featured Snaps: Skip (Select Done)
9. Finish install → Reboot → Remove USB

---

# 🔹 STEP 4 — UPDATE SYSTEM

Open Terminal:

```bash
sudo apt update && sudo apt upgrade -y
```

---

# 🔹 STEP 5 — SET STATIC IP (CRITICAL)

### Find network interface

```bash
ip a
```

Example interface: `enp0s3`

### Edit Netplan

```bash
sudo nano /etc/netplan/01-network-manager-all.yaml
```

Paste (change interface if needed):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s31f6:
      dhcp4: false
      addresses:
        - 192.168.5.25/24
      gateway4: 192.168.5.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Apply:

```bash
sudo netplan apply
```

Verify:

```bash
ip a
```

---

# � STEP 6 — CONFIGURE REMOTE ACCESS (SSH + XRDP)

## 📌 Environment
- OS: Ubuntu Server (22.04 / 24.04)
- GUI: XFCE
- Remote GUI Access: XRDP
- Remote CLI Access: SSH
- Network: Static IP / Authenticated Network

---

## 6.1 Create a New User

```bash
sudo adduser devuser
sudo usermod -aG sudo devuser
```

Verify:

```bash
id devuser
```

---

## 6.2 Enable SSH (CLI Access)

Install & start SSH

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```

Check status:

```bash
sudo systemctl status ssh
```

Connect from Windows

```bash
ssh devuser@SERVER_IP
```

Example:

```bash
ssh devuser@192.168.5.25
```

---

## 6.3 Install GUI (XFCE) + XRDP

```bash
sudo apt install xfce4 xfce4-goodies xrdp -y
```

Enable XRDP:

```bash
sudo systemctl enable xrdp
sudo systemctl start xrdp
```

Allow RDP port:

```bash
sudo ufw allow 3389/tcp
sudo ufw reload
```

---

## 6.4 Configure XRDP to Use XFCE (IMPORTANT)

Per-user XFCE session (REQUIRED for every user)

Run as that user:

```bash
echo "startxfce4" > ~/.xsession
chmod +x ~/.xsession
rm -f ~/.Xauthority
```

---

System-wide XRDP Fix (ONE-TIME)

Edit XRDP startup script:

```bash
sudo nano /etc/xrdp/startwm.sh
```

Comment this line:

```bash
# exec /bin/sh /etc/X11/Xsession
```

Add below it:

```bash
startxfce4 &
```

Restart XRDP:

```bash
sudo adduser xrdp ssl-cert
sudo systemctl restart xrdp
```

---

## 6.5 Connect to GUI from Windows (RDP)

1. Press Win + R
2. Type mstsc
3. Computer: SERVER_IP
4. Click Connect

XRDP Login:

Session: Xorg
Username: devuser
Password: user password

✅ XFCE desktop will load.

---

## 6.6 Fix “Oh no! Something has gone wrong” (NEW USERS)

This happens when .xsession is missing.

Fix:

```bash
sudo -u devuser bash
echo "startxfce4" > ~/.xsession
chmod +x ~/.xsession
rm -f ~/.Xauthority
exit
sudo systemctl restart xrdp
```

Apply to All Future Users (Recommended)

```bash
sudo nano /etc/skel/.xsession
```

Add:

```bash
startxfce4
```

---

## 6.7 Disable Sleep / Suspend (SERVER MODE)

Disable system sleep completely

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

---

Disable idle power actions

```bash
sudo nano /etc/systemd/logind.conf
```

Set:

```ini
HandleSuspendKey=ignore
HandleHibernateKey=ignore
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
IdleAction=ignore
IdleActionSec=0
```

Apply:

```bash
sudo systemctl restart systemd-logind
```

---

Disable XRDP idle timeout

```bash
sudo nano /etc/xrdp/sesman.ini
```

Set:

```ini
KillDisconnected=false
DisconnectedTimeLimit=0
IdleTimeLimit=0
```

Restart:

```bash
sudo systemctl restart xrdp
```

---

Disable Screen Blanking (XFCE)

Run as XRDP user:

```bash
xset -dpms
xset s off
xset s noblank
```

Make permanent:

```bash
nano ~/.xprofile
```

Add:

```bash
xset -dpms
xset s off
xset s noblank
```

---

## 6.8 Prevent SSH Idle Disconnect (Optional)

```bash
sudo nano /etc/ssh/sshd_config
```

Add:

```ini
ClientAliveInterval 60
ClientAliveCountMax 0
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

## 6.9 Safe Reboot with XRDP Users Logged In

Check sessions:

```bash
loginctl
```

Terminate XRDP user:

```bash
sudo loginctl terminate-user devuser
```

Force reboot (admin):

```bash
sudo reboot -f
```

---

## 6.10 Final Result (Remote Access)

✔ SSH access working
✔ GUI access via Windows RDP
✔ New users log in without crash
✔ XFCE stable (no GNOME issues)
✔ No sleep / suspend
✔ Server always ON

---

## 6.11 Notes

Use SSH for daily admin work
Use XRDP for GUI-only tasks
Recommended for lab / office / on-prem servers

---

# 🔹 STEP 7 — INSTALL REQUIRED SOFTWARE

```bash
sudo apt install -y \
nginx \
php php-fpm \
python3 python3-pip python3-venv \
git curl \
samba \
ufw
```

Install Node.js (for React):

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

---

# 🔹 STEP 8 — MYSQL CONFIGURATION

---

## 8.1 Install MySQL Server

```bash
sudo apt update
sudo apt install mysql-server -y
```

Check status:

```bash
sudo systemctl status mysql
```

---

## 8.2 Basic MySQL Security Setup

```bash
sudo mysql_secure_installation
```

Choose:

```
VALIDATE PASSWORD COMPONENT?  No
Set root password?           Yes
Remove anonymous users?      Yes
Disallow root login remotely? Yes
Remove test database?        Yes
Reload privilege tables?     Yes
```

---

## 8.3 Login as Root (Local Only)

```bash
sudo mysql -u root -p
```

---

## 8.4 Create Application User (No Database)

```sql
CREATE USER 'devadmin'@'%' IDENTIFIED BY 'Dev@123';
```

---

## 8.5 Grant Permissions (Application Creates DB)

```sql
GRANT CREATE, ALTER, DROP, INDEX, SELECT, INSERT, UPDATE, DELETE
ON *.* TO 'devadmin'@'%';

FLUSH PRIVILEGES;
EXIT;
```

---

## 8.6 Allow MySQL to Accept LAN Connections

Edit MySQL config:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Set:

```
bind-address = 0.0.0.0
```

Restart MySQL:

```bash
sudo systemctl restart mysql
```

---

## 8.7 Connect Using MySQL Workbench (Windows GUI)

### On Your Windows PC

1. Install **MySQL Workbench**
2. Open **MySQL Workbench**
3. Click **➕ New Connection**

### Enter the Following Details

```
Connection Name : Dev_Server
Hostname        : 192.168.5.25
Port            : 3306
Username        : devadmin
Password        : Dev@123
```

4. Click **Test Connection**
5. Save the connection

---

# 🔹 STEP 9 — CREATE PROJECT STRUCTURE

```bash
sudo mkdir -p /srv/dev/{php,flask,django}
sudo chown -R devadmin:www-data /srv/dev
sudo chmod -R 775 /srv/dev
```

---

# 🔹 STEP 10 — CONFIGURE STORAGE SERVER (SAMBA)

### Edit Samba config

```bash
sudo nano /etc/samba/smb.conf
```

Add at the bottom:

```ini
[dev]
path = /srv/dev
browseable = yes
writable = yes
create mask = 0775
directory mask = 0775
valid users = devadmin
```

Set Samba password:

```bash
sudo smbpasswd -a devadmin
```

Restart Samba:

```bash
sudo systemctl restart smbd
```

---

# 🔹 STEP 11 — MAP STORAGE AS DRIVE IN WINDOWS

### On Windows PC:

1. Open **File Explorer**
2. Right-click **This PC → Map network drive**
3. Choose drive letter (example **Z:**)
4. Folder:

```
\\192.168.5.25\dev
```

5. Check:

   * ☑ Reconnect at sign-in
6. Enter credentials:

   * Username: `devadmin`
   * Password: Dev@123

✅ Storage now appears as drive **Z:**

---

# 🔹 STEP 12 — PHP PROJECTS (DEV)

### Example PHP project

Create folder:

```
Z:\php\project1
```

Create `index.php`:

```php
<?php
echo "PHP Project 1 working";
```

---

# 🔹 STEP 13 — FLASK PROJECTS (DEV)

### Flask API

```bash
cd /srv/dev/flask
mkdir convex_msql
cd convex_msql
python3 -m venv venv
source venv/bin/activate
pip install flask gunicorn
```

Create `app.py`:

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Flask API working"
```

Run:

```bash
tmux new -s gunicorn-dev
cd /srv/dev/flask/convex_msql
source venv/bin/activate
gunicorn -b 127.0.0.1:5001 app:app --reload
Detach from tmux (IMPORTANT)Press: Ctrl + B, then D
Reattach later : tmux attach -t gunicorn-dev
kill session : tmux kill-session -t gunicorn-dev
```

---

# 🔹 STEP 14 — DJANGO PROJECTS (DEV)

```bash
cd /srv/dev/django
mkdir shop
cd shop
python3 -m venv venv
source venv/bin/activate
pip install django gunicorn
django-admin startproject mysite .
```

Run:

```bash
tmux new -s gunicorn-dev
cd /srv/dev/flask/convex_msql
source venv/bin/activate
gunicorn mysite.wsgi:application -b 127.0.0.1:8001 --reload
Detach from tmux (IMPORTANT)Press: Ctrl + B, then D
Reattach later : tmux attach -t gunicorn-dev
kill session : tmux kill-session -t gunicorn-dev
```

---

# 🔹 STEP 15 — NGINX CONFIG (PATH-BASED ROUTING)

```bash
sudo nano /etc/nginx/sites-available/dev
```

script:

```nginx
server {
    listen 80 default_server;
    server_name _;

    root /srv/dev/php;
    index index.php index.html;

    # Allow directory index resolution
    location / {
        try_files $uri $uri/ $uri/index.php =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }

   location /flask/convex_msql/ {
        proxy_pass http://127.0.0.1:5001/;
        include proxy_params;
    }
    # add config for every new flask,django,react,node applications and restart the nginx

  
}

```
restart nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

# � STEP 16 — ACCESS FROM WINDOWS (FINAL)

| Project | URL                                |
| ------- | ---------------------------------- |
| PHP     | `http://192.168.5.25/project1` |
| Flask   | `http://192.168.5.25/convex_msql/`   |
| Django  | `http://192.168.5.25/shop/` |
| Storage | `Z:\`                              |

---

# ✅ DAILY WORKFLOW (IMPORTANT)

1. Open **Z:** on Windows
2. Edit files in VS Code
3. Flask/Django auto-reload
4. Refresh browser
5. Done

No uploads.
No physical server access.

---

# 🧠 NOTES (READ)

* Always use **trailing slash** for Flask/Django paths
* One project = one Gunicorn port
* Dev server only (not production)

---
## updated verion will be on the web page "index.html"
