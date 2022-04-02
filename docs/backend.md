
# Deployment for Backend

## Project Installation Location
`/home/ubuntu/ta-management/`

## Operating System Requirements
- [Ubuntu 18.04.6 LTS](https://releases.ubuntu.com/18.04/)
- [Ubuntu 20.04.04 LTS](https://ubuntu.com/download/desktop)

---

## Step 1 — Installing Packages from the Ubuntu Repositories
#### Download the latest package information
`sudo apt update` 
#### Install these packages
`sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl` 


## Step 2 — Creating the PostgreSQL Database and User
#### Login to PostgreSQL
`sudo -u postgres psql` 
#### Create a database
`CREATE DATABASE myproject;` 
#### Create a database user for your project and select a secure password
`CREATE USER myprojectuser WITH PASSWORD 'password';` 
#### Set the default encoding to UTF-8 which Django expects
`ALTER ROLE myprojectuser SET client_encoding TO 'utf8';` 
#### Set the default transaction isolation scheme to read committed to block reads from uncommitted transactions
`ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';` 
#### Set the relevant timezone
`ALTER ROLE myprojectuser SET timezone TO 'UTC';` 
#### Give new user access to administer your new database
`GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;` 

## Step 3 — Creating a Python Virtual Environment using venv for your Project

#### Upgrade pip
`sudo -H pip3 install --upgrade pip` 

#### Move into this directory
`cd /home/ubuntu/ta-management/backend/ta_sys/` 

#### Create a Python virtual environment
`venv env` 

#### Activate the virtual environment
`source env/bin/activate` 

#### Install these Python packages in your virtual environment
`pip install django gunicorn psycopg2-binary` 

## Step 4 — Configuring the Django Project

#### Open settings.py
`sudo vim ~/ta-management/backend/ta_sys/ta_sys/settings.py`

#### Add the database configuration

```
settings.py

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'myproject',
        'USER': 'myprojectuser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

## Step 5 - Completing the Initial Project Setup
#### Make migrations
`(env) ~/ta-management/backend/ta_sys/manage.py makemigrations` 

#### Apply migrations
`(env) ~/ta-management/backend/ta_sys/manage.py migrate`

#### Create an administrative user for the project
`(env) ~/ta-management/backend/ta_sys/manage.py createsuperuser`

## Step 6 — Testing Gunicorn’s Ability to Serve the Project

#### Move into project directory
`(env) cd ta-management/backend/ta_sys`

#### Run gunicorn to load the project's WSGI module
`(env) gunicorn --bind 0.0.0.0:8000 ta_sys.wsgi` 

#### Deactivate the virtual environment
`(env) deactivate` 

## Step 7 — Creating systemd Socket and Service Files for Gunicorn


#### Create and open a systemd socket file
`sudo nano /etc/systemd/system/gunicorn.socket`

```
/etc/systemd/system/gunicorn.socket

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

#### Create and open a systemd service file for Gunicorn
`sudo nano /etc/systemd/system/gunicorn.service`

```
/etc/systemd/system/gunicorn.service

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/ta-management/backend/ta_sys
ExecStart=/home/ubuntu/ta-management/backend/ta_sys/env/bin/gunicorn \
	--access-logfile - \
	--workers 12 \
	--bind unix:/run/gunicorn.sock \
	ta_sys.wsgi:application

[Install]
WantedBy=multi-user.target
```

#### Start the Gunicorn Socket
`sudo systemctl start gunicorn.socket`

#### Enable the Gunicorn Socket
`sudo systemctl enable gunicorn.socket`

## Step 8 — Checking for the Gunicorn Socket File

#### Check the status of the process to find out whether it was able to start
`sudo systemctl status gunicorn.socket`
```
ubuntu@ta-management-final:~$ sudo systemctl status gunicorn.socket
   Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor preset: 
   Active: active (running) since Sun 2022-03-27 22:29:33 UTC; 5 days ago
   Listen: /run/gunicorn.sock (Stream)
   CGroup: /system.slice/gunicorn.socket

Mar 27 22:29:33 ta-management-final systemd[1]: Listening on gunicorn socket.
```

#### Check for the existence of the gunicorn.sock file within the /run directory
`file /run/gunicorn.sock`

```
Output

ubuntu@ta-management-final:~$ file /run/gunicorn.sock
/run/gunicorn.sock: socket
```

#### Check the Gunicorn socket’s logs
`sudo journalctl -u gunicorn.socket` 

```
Output

ubuntu@ta-management-final:~$ sudo journalctl -u gunicorn.socket
-- Logs begin at Sun 2022-03-27 08:36:42 UTC, end at Sat 2022-04-02 02:33:52 UTC
Mar 27 22:29:33 ta-management-final systemd[1]: Listening on gunicorn socket.
```

## Step 9 — Testing Socket Activation

#### Check the status
`sudo systemctl status gunicorn`

```
Output 

● gunicorn.service - gunicorn daemon
   Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
```

#### Test the socket activation mechanism by sending a connection to the socket
`curl --unix-socket /run/gunicorn.sock localhost`

#### Check the status again
`sudo systemctl status gunicorn`

```
Output

● gunicorn.service - gunicorn daemon
   Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2022-03-29 05:28:48 UTC; 3 days ago
 Main PID: 13446 (gunicorn)
    Tasks: 13 (limit: 4915)
   CGroup: /system.slice/gunicorn.service
           ├─13446 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13482 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13484 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13486 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13488 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13490 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13491 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13493 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13495 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13497 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13498 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           ├─13499 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/
           └─13500 /home/ubuntu/ta-management/backend/ta_sys/env/bin/python3 /home/ubuntu/ta-management/backend/ta_sys/env/

        ...
```

#### Check the logs for additional details if problem occurs
`sudo journalctl -u gunicorn`

#### After making any changes to gunicorn.service file
`sudo systemctl daemon-reload`

#### Restart the Gunicorn process
`sudo systemctl restart gunicorn`

## Step 10 - Configure Nginx to Proxy Pass to Gunicorn

#### Open a new server block in Nginx’s sites-available directory

`sudo nano /etc/nginx/sites-available/django`

#### Specify that the block should listen on port 80, the location of static assets, and information about the favicon

```
/etc/nginx/sites-available/django

server {
    listen 80 default_server;
	  listen [::]:80 default_server;
    
    server_name api.ua-compsci-ta.ca;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/ta-management/backend/ta_sys;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

#### Enable the file by linking it to the sites-enabled directory
`sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled`

#### Test the Nginx configuration for syntax errors
`sudo nginx -t`

```
Output

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

#### If there are no errors, restart Nginx
`sudo systemctl restart nginx`

#### Adjust the firewall settings by removing the rule to open port 8000
`sudo ufw delete allow 8000`

#### Allow normal traffic on ports 80 and 443
`sudo ufw allow 'Nginx Full'`

---

## For updates to Django Application
`sudo systemctl restart gunicorn`

## For changes in Gunicorn socket or service files
```
sudo systemctl daemon-reload
sudo systemctl restart gunicorn.socket gunicorn.service
```

## For changes in Nginx server block configuration
`sudo nginx -t && sudo systemctl restart nginx`

## Reference

[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04)

