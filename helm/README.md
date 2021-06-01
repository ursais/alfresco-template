# Operations

## Table of Contents
* [Prerequisites](#Prerequisites)
* [UFW](#UFW)
* [Nginx](#Nginx)
* [Systemd](#Systemd)
* [Release](#Release)

## Prerequisites

```shell
apt install certbot docker-compose git nginx python3-certbot-nginx ufw
cd /opt
git clone https://github.com/ursais/alfresco-template alfresco
```

## UFW

```shell
ufw allow ssh
ufw allow http
ufw allow https
ufw enable
```

## Nginx

* Create `/etc/nginx/sites-available/alfresco.example.com` with:

```nginx
upstream alfresco {
    server 127.0.0.1:8080; 
}

server {
    listen 80;
    server_name alfresco.example.com;

    location / {
        proxy_pass  http://alfresco;
    }
}
```  

* Enable, configure and start Nginx

```shell
cd /etc/nginx/sites-enabled
ln -s ../sites-available/alfresco.example.com .
nginx -t
certbot 
```

## Systemd

Create `/etc/systemd/system/alfresco.service` with:

```unit file (systemd)
[Unit]
Description=Alfresco container starter
After=docker.service network-online.target
Requires=docker.service network-online.target

[Service]
WorkingDirectory=/opt/alfresco
Type=oneshot
RemainAfterExit=yes

ExecStartPre=-/usr/local/bin/docker-compose pull --quiet
ExecStart=/usr/local/bin/docker-compose -f docker-compose.yml up -d

ExecStop=/usr/local/bin/docker-compose -f docker-compose.yml down

ExecReload=/usr/local/bin/docker-compose pull --quiet
ExecReload=/usr/local/bin/docker-compose -f docker-compose.yml up -d

[Install]
WantedBy=multi-user.target
```

and run:
```shell
systemctl daemon-reload
systemctl enable template
service template start
```

## Release

Run:
```shell
cd /opt/alfresco
git pull
service alfresco restart
```
