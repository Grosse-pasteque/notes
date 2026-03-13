# How to setup your own home server ?

### 1 - Requirements

- a machine that can stay on 24/7, preferably `linux server` or a server system
- `nginx` installed (apt)
```bash
sudo apt install nginx
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

# 2 - Linking your ip to the domain you own

```bash
sudo certbot certonly --webroot -w /var/www/certbot -d SITE.com
```
**NOTE:** _Auto renews certific_

# 3 - Create a nginx config

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
```bash
sudo nginx -t
sudo systemctl reload nginx
```

# 4 - System process setup

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

# 5 - Example node js server

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
