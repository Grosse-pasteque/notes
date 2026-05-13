# How to setup your own home server ?

### 1 - Requirements

- a machine that can stay on 24/7, preferably `linux server` or a server system
- `nginx` installed (apt)
```bash
sudo apt install nginx certbot
sudo systemctl enable nginx
sudo systemctl start nginx
```
- `nginx` allowed throught your device firewall
```bash
sudo ufw allow 'Nginx Full'
sudo ufw reload
```
or your firewall disabled
```bash
sudo ufw disable
```
- owning a domain with an A Record created pointing to your ip
- your router forwarding ports 443 and 80 to your machine
- your router firewall allowing ports 443 and 80

# 2 - Setup nginx to pass ACME challenge

```bash
sudo nano /etc/nginx/sites-available/default
```
Minimal default config
```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name _;

    location ^~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Reload nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```

# 3 - Linking your ip to the domain you own

```bash
sudo certbot certonly --webroot -w /var/www/certbot -d SITE.com
```
**NOTE:** _Auto renews certificate_

Your certificates renewal can be tested via
```bash
sudo certbot renew --dry-run
```

# 4 - Create a nginx config

```bash
sudo nano /etc/nginx/sites-available/SITE.conf
```
```nginx
server {
    listen 443 ssl;
    server_name SITE.com;

    # https support (optional)
    ssl_certificate /etc/letsencrypt/live/SITE.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/SITE.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;

        # ip forwarding
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # socket support (optional)
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }
}
```
A simpler route can just be
```nginx
server {
    # ...

    location / {
        default_type text/plain;
        return 200 "hello world";
   }
   location /test {
        default_type application/json;
        return 200 "[1, 2, 3]";
   }
}
```
Enable your site config
```bash
sudo ln -s /etc/nginx/sites-available/SITE.conf /etc/nginx/sites-enabled/
```
Reload nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```

# 5 - System process setup

Having your app as a system process makes it auto restart on crash.

```bash
sudo nano /etc/systemd/system/SITE.service
```
```ini
Description=Hello Word

[Service]
ExecStart=/usr/bin/node /home/server
Restart=always
User=root
Environment=NODE_ENV=production
WorkingDirectory=/home/server/

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable SITE
sudo systemctl start SITE
journalctl -u SITE -n 10
```

# 6 - Example node js server

Nginx forwards traffix directly to 127.0.0.1:8080 so you don't need to get the certificates on your node app (using https package).
Hence the server config for when debugging and when hosting the final app doesn't change.
Making things way easier.

```js
const http = require('http');
const express = require('express');

const app = express();
const server = http.createServer(app);

app.set('trust proxy', true);
app.use('/', (req, res) => console.log(req.ip));

server.listen(8080, () => console.log('listening...'));
```

# 7 - Security

When you expose your ip your server becomes very vulnerable (scrappers, ssh brute force, bots, exploits, ...)
To fix this you can:
- disable ssh password login
- disable ssh root login
```bash
sudo nano /etc/ssh/sshd_config
```
```conf
PasswordAuthentication no
ChallengeResponseAuthentication no
KbdInteractiveAuthentication no
AuthenticationMethods publickey
UsePAM no
```
Find overwrites and change them
```bash
grep -Ri "PasswordAuthentication" /etc/ssh/
```

- change ssh default port (22) to something else: 
it can be changed in the ssh config or change the external port that you forward on your router panel
- setup fail2ban to blacklist ips that bruteforce ssh password
```bash
sudo apt install fail2ban
sudo nano /etc/fail2ban/jail.local
```
```ini
[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
backend = systemd

maxretry = 3
findtime = 600
bantime = 3600
```
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```
