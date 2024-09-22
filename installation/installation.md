# installation and provisioning

## 1. prerequisites

The assumption is that we start from a VM (VPS) with ubuntu 22.04 or newer installed.

You need also to purchase a fqdn pointing to the IP address of the VM (VPS).  Optionally you can purchase a SSL certificate or otherwise let's a generate/renew via let's encrypt

## 2. VM preparation

To be done only once after setting up/requesting the VM or VPS

```bash
sudo -i  # if not logged in a root
apt-get update
apt-get -y upgrade
apt-get install -y samba smbclient cifs-utils certbot nginx pip python3 ffmpeg
python3 -m pip config set global.break-system-packages true
```

## 3. Setup the site

A site is determined by the combination of fqdn/port which has to be unique per site.  One site is one to one related with one database.

Setup first a home user for the site (please replace site1 with the real name)

```bash
# make sure you are root
adduser site1
visudo
```

Add following text in visudo session and save
```text
# User privilege specification
root    ALL=(ALL:ALL) ALL
site1   ALL=(ALL:ALL) ALL
```
Still as root, add site1 to the www-data group and add static directory + set permissions

```bash
gpasswd -a www-data site1
mkdir -p /home/site1/static/coach
mkdir -p /home/site1/static/video
mkdir -p /home/site1/static/app
chmod g+x /home/site1  && chmod 777 /home/site1/static
chmod 777 /home/site1/static/coach/  && chmod 777 /home/site1/static/video/ && chmod 777 /home/site1/static/app/
```

Add SSL cert, clone repo and install pip dependencies

```bash
su - site1
ssh-keygen
cd .ssl
# provide public certificate (.pub file) in github so cloning can take place
cd
git clone git@github.com:sprenge/kineperfectserver.git
git clone git@github.com:sprenge/kineperfectclient.git
cd kineperfectserver/kineperfect/backend
# following steps only for the first site
sudo pip install -r requirements.txt --break-system-packages
sudo pip install gunicorn  --break-system-packages

```

Provision static files

```bash
# to be executed as site1 user
cd
cp -r kineperfectclient/coach/dist/* ~/static/coach
cp -r tbc ~/static/app
cd kineperfectserver/kineperfect
python3 manage.py collectstatic
python3 manage.py migrate
```

Provision database from a backup

```bash
python3 manage.py loaddata backup.json
```


Define in nginx config 

as a root user edit /etc/nginx/sites-enabled/default and create following section
```bash
server {
        ssl on;
        listen 8130 ssl default_server;
        listen [::]:8130 ssl default_server;
        ssl_certificate /etc/letsencrypt/live/kineperfect.duckdns.org/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/kineperfect.duckdns.org/privkey.pem;
        server_name kineperfect.duckdns.org;

        location /static/ {
            root /home/site1;
        }
        location / {
            proxy_pass http://unix:/run/gunicorn_site1.sock;
            proxy_pass_header Server;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-Protocol $scheme;
            proxy_set_header    Upgrade     $http_upgrade;
            proxy_set_header    Connection  "upgrade";
            client_max_body_size 500M;
        }
}

```

```bash
systemctl restart nginx
```
Gunicorn service, please replace site1 with your site name

```bash
sudo -i # make sure to be root
mkdir /run/gunicorn_site1
chown site1:www-data /run/gunicorn_site1
chmod 770 /run/gunicorn_site1


cat <<EOF > /etc/systemd/system/gunicorn_site1.service
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=site1
Group=www-data
WorkingDirectory=/home/site1/kineperfectserver/kineperfect
ExecStart=/usr/local/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --time 600 \
          --bind unix:/run/gunicorn_site1/gunicorn_site1.sock \
          kineperfect.wsgi:application

[Install]
WantedBy=multi-user.target
EOF
systemctl start gunicorn.service