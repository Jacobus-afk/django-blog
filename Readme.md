# Djang blog

Corey Shafer's django tutorials: https://www.youtube.com/playlist?list=PL-osiE80TeTtoQCKZ03TU5fNfx2UY6U4p

## ubuntu server config

* ssh root@`<server ip>`
* apt-get update && apt-get upgrade
* hostnamectl set-hostname `<hostname>`
* nano /etc/hosts, add `<server ip>` `<hostname>`
* adduser `<username>`
* adduser `<username>` sudo
* exit
* ssh `<username>`@`<server ip>`
* mkdir -p ~/.ssh

* open local terminal and follow ssh instructions under **ssh config**

* sudo chmod 700 ~/.ssh/
* sudo chmod 600 ~/.ssh/*
* exit
* ssh `<username>`@`<server ip>`
* sudo nano /etc/ssh/sshd_config - PermitRootLogin no, PasswordAuthentication no
* sudo systemctl restart sshd

* sudo apt-get install ufw
* sudo ufw default allow outgoing
* sudo ufw default deny incoming
* sudo ufw allow ssh
* sudo ufw allow 8000 (for testing)
* sudo ufw enable
* sudo ufw status

* open local terminal and follow **pushing project to server** instructions

* sudo apt-get install python3-pip
* sudo apt-get install python3-venv
* cd ~/`<django project>`
* python3 -m venv ~/`<django project>`/venv
* source ~/`<django project>`/venv/bin/activate
* pip install -r ~/`<django project>`/requirements.txt
* sudo nano ~/`<django project>`/django_project/settings.py - ALLOWED_HOSTs = ["`<server ip>`"], STATIC_ROOT = os.path.join(BASE_DIR, 'static')
* python manage.py collectstatic

* sudo apt-get install apache2
* sudo apt-get install libapache2-mod-wsgi-py3
* sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/django_project.conf
* sudo nano /etc/apache2/sites-available/django_project.conf add 

```
  Alias /static /home/YOURUSER/YOURPROJECT/static
  <Directory /home/YOURUSER/YOURPROJECT/static>
    Require all granted
  </Directory>

  Alias /media /home/YOURUSER/YOURPROJECT/media
  <Directory /home/YOURUSER/YOURPROJECT/media>
    Require all granted
  </Directory>

  <Directory /home/YOURUSER/YOURPROJECT/YOURPROJECT>
    <Files wsgi.py>
      Require all granted
    </Files>
  </Directory>

  WSGIScriptAlias / /home/YOURUSER/YOURPROJECT/YOURPROJECT/wsgi.py
  WSGIDaemonProcess django_app python-path=/home/YOURUSER/YOURPROJECT python-home=/home/YOURUSER/YOURPROJECT/venv
  WSGIProcessGroup django_app
```
* sudo a2ensite django_project.conf
* sudo a2dissite 000-default.conf
* sudo chown :www-data `<django project>`/db.sqlite3
* sudo chmod 664 `<django project>`/db.sqlite3
* sudo chown :www-data `<django project>`/
* sudo chown -R :www-data `<django project>`/media/
* sudo chmod -R 775 `<django project>`/media
* sudo touch /etc/config.json
* `<secret key>`: python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
* or on console:
```
python
import secrets
secrets.token_hex(24)
```
* sudo nano /etc/config.json
```
{
        "SECRET_KEY": "<secret key>",
        "EMAIL_USER": "<email address>",
        "EMAIL_PASS": "<generated app key>"
}
```

* sudo nano `<django project>`/django_project/settings.py
```
import json

with open('/etc/config.json') as config_file:
    config = json.load(config_file)

SECRET_KEY = config['SECRET_KEY']
DEBUG = False

EMAIL_HOST_USER = config['EMAIL_USER']
EMAIL_HOST_PASSWORD = config['EMAIL_PASS']
```
* sudo ufw delete allow 8000
* sudo ufw allow http/tcp

* systemctl reload apache2
## ssh config 

* ssh-keygen -b 4096
* scp \~/.ssh/`<keyname>`.pub `<username>`@`<server ip>`:\~/.ssh/authorized_keys

## pushing project to server

* activate environment
* pip freeze > requirements.txt
* scp -r .\\`<django project>`\  `<username>`@`<serverver ip>`:~/

## running server
* python manage.py runserver 0.0.0.0:8000

## enabling https with ssl/tls certificate

* certbot
```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-apache
```
* sudo nano /etc/apache2/sites-available/django_project.conf - ServerName www.`<domain name>`, comment out the WSGI entries
* sudo certbot --apache
* sudo nano /etc/apache2/sites-available/django_project.conf - Remove entries starting with Alias and ending with commented out WSGI
* sudo nano /etc/apache2/sites-available/django_project-le-ssl.conf - Uncomment out WSGI lines
* sudo certbot renew --dry-run
* sudo ufw allow https
* sudo service apache2 restart

## auto renewing certificates

* sudo crontab -e
```
30 4 1 * * sudo certbot renew --quiet
```
