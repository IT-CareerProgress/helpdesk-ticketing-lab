# Help Desk Ticketing Lab: osTicket on Ubuntu Server

A self-hosted help desk ticketing system built in a virtual machine on an 8 GB laptop with a $0 budget. I deployed the full web stack (Apache, MariaDB, PHP), installed osTicket, secured it, and used it to work a help desk ticket from intake to resolution.

## Environment

| Component | Detail |
|---|---|
| Host | Acer laptop, 8 GB RAM, Windows |
| Hypervisor | Oracle VirtualBox |
| Guest OS | Ubuntu Server, headless (no desktop) |
| Web server | Apache 2 |
| Database | MariaDB 11.8.6 |
| Runtime | PHP |
| Application | osTicket v1.18.4 |
| Networking | NAT with port forwarding |
| Admin access | SSH from Windows PowerShell |

## Architecture

```
Windows host (Chrome / PowerShell)
      |  127.0.0.1:8080 (web)     127.0.0.1:2222 (SSH)
      v
VirtualBox NAT port forwarding
      v
Ubuntu Server VM (10.0.2.15)
  Apache (80) -> PHP -> osTicket -> MariaDB (database: osticket)
  OpenSSH (22)
```

A browser request reaches Apache, Apache passes the PHP files to PHP, and PHP stores tickets in MariaDB through a limited-privilege database account.

## Setup

### 1. Create the VM
Created an Ubuntu (64-bit) VM in VirtualBox with about 2 GB RAM, 1 CPU, and a 20 GB dynamically allocated disk. Confirmed virtualization was enabled in Task Manager first. Installed Ubuntu Server from the ISO using the whole virtual disk and selected the OpenSSH server option.

### 2. Configure networking
Used **NAT** so the VM can reach the internet while hidden behind the host, with **port forwarding** to reach it from Windows:

| Name | Protocol | Host port | Guest port |
|---|---|---|---|
| SSH | TCP | 2222 | 22 |
| HTTP | TCP | 8080 | 80 |

High host ports were used because low ports can be restricted or already in use on the host. Verified inside the VM:

```
ip -br a     # enp0s3 had 10.0.2.15
free -h      # about 415 MiB used of 1.6 GiB
```

### 3. Enable SSH and update the system
SSH was installed but stopped (see Problems below). Fixed with:

```
sudo systemctl enable --now ssh
sudo apt update
sudo apt upgrade -y
```

From then on I administered the server from PowerShell: `ssh -p 2222 <username>@127.0.0.1`

### 4. Install the LAMP stack

```
sudo apt install -y apache2 mariadb-server php libapache2-mod-php php-mysql php-gd php-mbstring php-intl php-apcu php-xml
```

I ran `php -m` to see which modules were already present and installed only what was missing. Verified Apache by loading its default page at `http://127.0.0.1:8080`, then confirmed PHP worked through Apache with a temporary `phpinfo()` page, which I deleted afterward because it exposes server details.

### 5. Create the database and a least-privilege user

```sql
CREATE DATABASE osticket CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'osticket_user'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON osticket.* TO 'osticket_user'@'localhost';
FLUSH PRIVILEGES;
SHOW GRANTS FOR 'osticket_user'@'localhost';
```

The application gets its own account, limited to one database and only from localhost. If the web app were ever compromised, the attacker would reach that database and not the whole server.

### 6. Install osTicket

```
cd /tmp
wget <osTicket v1.18.4 zip URL from github.com/osTicket/osTicket/releases>
sudo apt install -y unzip wget
unzip osTicket-v1.18.4.zip
sudo mkdir /var/www/html/osticket
sudo cp -r upload/. /var/www/html/osticket/
sudo cp /var/www/html/osticket/include/ost-sampleconfig.php /var/www/html/osticket/include/ost-config.php
sudo chown -R www-data:www-data /var/www/html/osticket
```

Only the `upload` folder goes on the web server, and the `/.` copies its contents instead of nesting the folder. I then ran the web installer at `http://127.0.0.1:8080/osticket/setup/` and connected it to the database with the limited account.

### 7. Harden the install

```
sudo rm -r /var/www/html/osticket/setup
sudo chmod 0444 /var/www/html/osticket/include/ost-config.php
```

The `setup` folder is removed so nobody can re-run the installer against the database. The config file holds the database credentials, so it is made read-only (`-r--r--r--`).

### 8. Verify
- Client portal loads at `http://127.0.0.1:8080/osticket/`
- Staff control panel loads at `http://127.0.0.1:8080/osticket/scp/` and the admin login works
- A test ticket was created from the portal and appeared in the staff queue

## Problems encountered

| Problem | Cause | Fix |
|---|---|---|
| SSH from PowerShell: "Connection reset", then "Connection refused" | Port forwarding was correct, but `systemctl status ssh` showed **inactive (dead)**, so nothing was listening on port 22 | `sudo systemctl enable --now ssh` |
| `ip a` showed only `127.0.0.1` | I was reading the loopback entry, which every machine has | Used `ip -br a` and found `10.0.2.15` on `enp0s3` |
| Terminal stuck on `END` and echoing "log file" | A pager (`less`) was open, reading my typing as pager commands | Pressed `q`; use `--no-pager` on status commands |
| Installer database error risk: `SHOW GRANTS` showed only `USAGE` | The user existed but had no privileges on the database | Ran the `GRANT ALL PRIVILEGES ON osticket.*` statement |
| `ERROR 1064` in MariaDB | Ended a statement with a colon, so MariaDB waited at the `->` prompt and merged my next line | Re-entered it with a semicolon |
| `php-imap` failed to install | Package not available from the repositories I was using | Skipped it; it only affects fetching email into tickets |

## Limitations

- No mail server is configured, so osTicket cannot send email notifications.
- The PHP IMAP extension is not installed, so tickets cannot be created from incoming email.
- While replying to a ticket I got an "unable to lock ticket" error and did not troubleshoot it further.
- There is no Active Directory in this lab.

## Ticket walkthrough

- [Ticket #670659: Can't log in (simulated)](tickets/001-cant-log-in.md)
