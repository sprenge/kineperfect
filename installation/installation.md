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
gpasswd -a www-data quinn
chmod g+x /home/site  && chmod g+x /home/site/static
chmod g+x /home/site/static/coach/  && chmod g+x /home/site/static/video/
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
# cp -r kineperfectclient/coach/dist/* ~/static/coach
# cp -r tbc ~/static/app
cd kineperfectserver/kineperfect
python3 manage.py collectstatic
```

Provision database from a backup

Provision a new database


Gunicorn service, please replace site1 with your site name

```bash
sudo -i # make sure to be root
cat <<EOF > /etc/systemd/system/gunicorn_site1.service
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=site1
Group=www-data
WorkingDirectory=/home/site1/kineperfectserver/kineperfect
ExecStart=/usr/local/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --time 600 \
          --bind unix:/run/gunicorn.sock \
          kineperfect.wsgi:application

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl start gunicorn.service