# Django React App Deployment in EC2 with Ngnix and Gunicorn

### What I start with<hr>
My project directory: `myprojectdir`<br>
My project root: `notionClone`<br>
My virtualenv: `env-nc`<br>
My FQDN: `test-notion-clone-tobi.inteam.jp`<br>
My ec2 instance user: `ubuntu`<br>


### Login to ec2 instance<hr>
`ssh -i C:\Users\monir\Downloads\YOUR-PEM-FILE.pem ubuntu@ec2-15-152-115-101.ap-northeast-3.compute.amazonaws.com`<br>


### Install essential libraries<hr>
`sudo apt update`<br>
`sudo apt install python3-pip python3-dev libpq-dev nginx curl`<br>


### Install different version of python according to our needs<hr>
`sudo add-apt-repository ppa:deadsnakes/ppa`<br>
`sudo apt-get update`<br>
`sudo apt install python3.7-distutils`<br>
`sudo apt-get install python3.7`<br>

### To check how many versions of python machine has<hr>
`ls -ls /usr/bin/python*`<br>

### Install pip and virtualenv utilities<hr>
`sudo -H pip3 install --upgrade pip`<br>
`sudo -H pip3 install virtualenv`<br>

### Create a project directory and change directory<hr>
`mkdir ~/myprojectdir`<br>
`cd ~/myprojectdir`<br>

### Create a virtualenv amd activate it<hr>
`virtualenv env-nc`<br>
`source env-nc/bin/activate`<br>


### Meanwhile --> Upload all the necessary folders within 'myprojectdir' in aws (/home/ubuntu/myprojectdir/)
### Meanwhile --> Edit inbound rules in aws security section (e.g. allow 'TCP', 'HTTP', 'HTTPS' etc.)

### Install necessary packages through pip<hr>
`env-nc: pip install -r requirements.txt`<br>

### Migrate database (db.sqlite3)<hr>
`env-nc: python manage.py migrate`<br>

### Allow firewall: port 8000<hr>
`env-nc: sudo ufw allow 8000`<br>

### Run the server and check in the browser if site starts working<hr>
`env-nc: python manage.py runserver 0.0.0.0:8000`<br>

### Binding gunicorn<hr>
`env-nc: gunicorn --bind 0.0.0.0:8000 notionClone.wsgi`<br>

### Collect stactic files<hr>
`env-nc: python manage.py collectstatic`<br>

### Deactivate virtualenv<hr>
`deactivate`<br>

### Creating systemd Socket File<hr>
`sudo nano /etc/systemd/system/gunicorn.socket`<br>

### Paste the following piece of configuration<hr>

```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

### Creating systemd Service File<hr>
`sudo nano /etc/systemd/system/gunicorn.service`<br>

### Paste the following piece of configuration for service creation<hr>

```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/myprojectdir
ExecStart=/home/ubuntu/myprojectdir/env-nc/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          notionClone.wsgi:application

[Install]
WantedBy=multi-user.target
```

### Gunicorn socket starting, enabling<hr>
`sudo systemctl start gunicorn.socket`<br>
`sudo systemctl enable gunicorn.socket`<br>

### Checking gunicorn status<hr>
`sudo systemctl status gunicorn.socket`<br>


### We can now check for the existence of the 'gunicorn.sock' file within the /run directory:<hr>
`file /run/gunicorn.sock`<br>


### If the systemctl status command indicated that an error occurred or if you do not find the gunicorn.sock file in the directory, 
It’s an indication that the Gunicorn socket was not able to be created correctly. Check the Gunicorn socket’s logs by typing:<hr>
`sudo journalctl -u gunicorn.socket`<br>


### Testing socket activation<hr>
`sudo systemctl status gunicorn`<br>


### Configure nginx to proxy pass to gunicorn<hr>
`sudo nano /etc/nginx/sites-available/notionClone`<br>


### Paste the following piece of configuration for ngnix<hr>

```
server {
    listen 80;
    server_name test-notion-clone-tobi.inteam.jp;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/myprojectdir;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

### Now, we can enable the file by linking it to the sites-enabled directory:<hr>
`sudo ln -s /etc/nginx/sites-available/notionClone /etc/nginx/sites-enabled`<br>


### Testing ngnix<hr>
`sudo nginx -t`<br>


### Restart nginx<hr>
`sudo systemctl restart nginx`<br>


### Finally, we need to open up our firewall to normal traffic on port 80. 
Since we no longer need access to the development server, we can remove the rule to open port 8000 as well:<hr>
`sudo ufw delete allow 8000`<br>
`sudo ufw allow 'Nginx Full'`<br>

### To allow db.sqlite3<hr>

`chmod 664 db.sqlite3`<br>
`sudo chown :www-data db.sqlite3`<br>
`sudo chown :www-data ~/myprojectdir/notionClone`<br>
`sudo sudo systemctl restart nginx`<br>

-------------done------------


### =======================OTHER USEFUL COMMANDS=================

### To delete ngnix sites-avialable, sites-enable rule<hr>
`sudo rm -r /etc/nginx/sites-available/notionClone`<br>
`sudo rm -r /etc/nginx/sites-enabled/notionClone`<br>


### Restart gunicorn (if needed)<hr>
`sudo systemctl daemon-reload`<br>
`sudo systemctl restart gunicorn`<br>


### To avoid psycopg2 error<hr>
`sudo apt-get install python3.7-dev`<br>
