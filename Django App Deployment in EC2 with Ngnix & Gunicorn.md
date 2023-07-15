# Django App Deployment in EC2 with Ngnix and Gunicorn

### 1. What I start with<hr>
Project root directory: `myprojectdir` [can hold multiple projects]<br>
Project root folder: `notebook_django_backend`<br>
Project main app: `backend` [inside 'notebook_django_backend']<br> 
My virtualenv: `env-notebook`<br>
My FQDN: `notebook-beta.inteam.jp`<br>
My ec2 instance user: `ubuntu`<br>

![image](https://user-images.githubusercontent.com/47719314/225372091-a1a86c18-f212-478c-8fb3-c1f7c89716aa.png)

👉 requirements.txt, manage.py files, main app will reside inside `notebook_django_backend` folder.

### 2. Login to ec2 instance<hr>
#### With CMD / Shell<br>
```
ssh -i C:\Users\monir\Downloads\YOUR-PEM-FILE.pem ubuntu@ec2-15-xxx-xxx-xxx.ap-xxx-3.compute.amazonaws.com
```

#### With Mobaxterm:<br>
Ctrl+Shift+N (will open up SSH session dialogue)<br>
Enter Remote host [ec2-xxx-xxx-xxx-118.ap-xxxx-1.compute.amazonaws.com] <br>
Click on Advance SSH Settings<br>
Check Use private key and upload .pem file

![image](https://user-images.githubusercontent.com/47719314/223616489-88ab1fcb-8874-48d4-918c-86bca7965c65.png)

### 3. Install essential libraries<hr>
```
sudo apt update
```
```
sudo apt install python3-pip python3-dev libpq-dev nginx curl
```

### 4. Install different version of python; I needed 3.7 python version for this specific project<hr>
```
sudo add-apt-repository ppa:deadsnakes/ppa
```
```
sudo apt-get update
```
```
sudo apt install python3.7-distutils
```
```
sudo apt-get install python3.7
```

### 5. To check how many python versions the machine has<hr>
```
ls -ls /usr/bin/python*
```

### 6. Install pip and virtualenv utilities<hr>
```
sudo -H pip3 install --upgrade pip
```
```
sudo -H pip3 install virtualenv
```

### 7. Create a project directory and change directory<hr>
```
mkdir ~/myprojectdir
```
```
cd ~/myprojectdir
```

### 8. Create a virtualenv and activate it<hr>
#### To create virtual enviornment with a specific version of python<br>
##### [ubuntu@ip-172-xx-xx-xx:~/myprojectdir$]
```
virtualenv --python=/usr/bin/python3.7 env-notebook
```

#### Or to create virtual enviornment with default version of python of the machine<br>
```
virtualenv env-notebook
```

#### Optional: For windows, if virtualenvwrapper is used, we can create env with specific python version <br>
```
pip install virtualenvwrapper-win
mkvirtualenv --python=python3.8 myenv
```



#### To activate<br>
##### [ubuntu@ip-172-xx-xx-xx:~/myprojectdir$]
```
source env-notebook/bin/activate
```

### 9. Meanwhile --> Upload all the necessary folders within 'myprojectdir' in aws (/home/ubuntu/myprojectdir/)
### 10. Meanwhile --> Edit inbound rules in aws security section (e.g. allow 'TCP', 'HTTP', 'HTTPS' etc.)


### 11. Install necessary packages through pip<hr>
##### [(env-notebook) ubuntu@ip-172-31-37-35:~/myprojectdir/notebook_django_backend$]
```
pip install -r requirements.txt
```

### 📢 To avoid psycopg2 error<hr>
`sudo apt-get install python3.7-dev`<br>


### 12. Migrate database<hr>
##### [(env-notebook) ubuntu@ip-172-31-37-35:~/myprojectdir/notebook_django_backend$]
```
python manage.py migrate
```

### 13. Allow firewall: port 8000<hr>
##### [(env-notebook) ubuntu@ip-172-31-37-35:~/myprojectdir/notebook_django_backend$]
```
sudo ufw allow 8000
```

### 14. Run the server and check in the browser if site starts working<hr>
##### [(env-notebook) ubuntu@ip-172-31-37-35:~/myprojectdir/notebook_django_backend$]
```
python manage.py runserver 0.0.0.0:8000
```

### 15. Binding gunicorn<hr>
##### [(env-notebook) ubuntu@ip-172-31-37-35:~/myprojectdir/notebook_django_backend$]
if requirements.txt has `gunicorn` then<br>
```
gunicorn --bind 0.0.0.0:8000 backend.wsgi
```

if requirements.txt does not have `gunicorn`, then we need to install `gunicorn`(with any version preference)<br>
```
pip install gunicorn 
```

### 16. Collect stactic files<hr>
##### [(env-notebook) ubuntu@ip-172-31-37-35:~/myprojectdir/notebook_django_backend$]
```
python manage.py collectstatic
```

### 17. Deactivate virtualenv<hr>
##### [(env-notebook) ubuntu@ip-172-31-37-35:~/myprojectdir/notebook_django_backend$]
```
deactivate
```

### 18. Creating systemd 'Socket File'<hr>
##### [ubuntu@ip-172-31-37-35:~/myprojectdir$]
```
sudo nano /etc/systemd/system/gunicorn.socket
```

#### Paste the following piece of configuration

```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

### 20. Creating systemd 'Service File'<hr>
```
sudo nano /etc/systemd/system/gunicorn.service
```

#### Paste the following piece of configuration for service creation

```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/myprojectdir/{root folder name, e.g. notebook_django_backend}
ExecStart=/home/ubuntu/myprojectdir/env-notebook/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          {main app name e.g. 'backend' which has wsgi file}.wsgi:application

[Install]
WantedBy=multi-user.target
```

### 21. Gunicorn socket starting, enabling<hr>
```
sudo systemctl start gunicorn.socket
```
```
sudo systemctl enable gunicorn.socket
```

### 22. Checking gunicorn.socket status<hr>
```
sudo systemctl status gunicorn.socket
```

### 📢 Restart gunicorn (when needed)<hr>
```
sudo systemctl daemon-reload
```
```
sudo systemctl restart gunicorn
```


#### We can now check for the existence of the 'gunicorn.sock' file within the /run directory:<hr>
`file /run/gunicorn.sock`<br>


#### If the systemctl status command indicated that an error occurred or if you do not find the gunicorn.sock file in the directory, 
It’s an indication that the Gunicorn socket was not able to be created correctly. Check the Gunicorn socket’s logs by typing:<hr>
```
sudo journalctl -u gunicorn.socket
```


### 23. Testing gunicorn status<hr>
```
sudo systemctl status gunicorn
```


### 24. Configure nginx to proxy pass to gunicorn<hr>
```
sudo nano /etc/nginx/sites-available/backend
```
👉 NOTE: I named it `backend`; it could be any name.

#### Paste the following piece of configuration for ngnix
```
server {
    listen 80;
    server_name notebook-beta-backend.inteam.jp;

    location = /favicon.ico { access_log off; log_not_found off; }

    location /static/ {
        alias /home/ubuntu/myprojectdir/notebook_django_backend/staticfiles/;
    }

    location /media/ {
        alias /home/ubuntu/myprojectdir/notebook_django_backend/media/;
    }


    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

### 25. Now, we can enable the file by linking it to the sites-enabled directory:<hr>
```
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled
```


### 26.Testing ngnix<hr>
```
sudo nginx -t
```


### 27. Restart nginx<hr>
```
sudo systemctl restart nginx
```

### 28. To permit static, media files <hr>
```
sudo usermod -a -G ubuntu www-data
```

### 29. Finally, we need to open up our firewall to normal traffic on port 80. 
Since we no longer need access to the development server, we can remove the rule to open port 8000 as well:<hr>
```
sudo ufw delete allow 8000
```
```
sudo ufw allow 'Nginx Full'
```


-------------done------------


### =======================OTHER USEFUL COMMANDS=================

### 👀 To allow db.sqlite3<hr>

```
chmod 664 db.sqlite3
```
```
sudo chown :www-data db.sqlite3
```
```
sudo chown :www-data ~/myprojectdir/backend
```
```
sudo sudo systemctl restart nginx
```

### 👀 To delete ngnix sites-avialable, sites-enable rule<hr>
`sudo rm -r /etc/nginx/sites-available/backend`<br>
`sudo rm -r /etc/nginx/sites-enabled/backend`<br>


### 🔥 JWT related error !<hr>

#### Error:
when i hit this without token: http://notebook-beta.inteam.jp/collection/list
it gives me a list perfectly.
when I hit anything URL with token, I am getting the following error
<head> <meta http-equiv="content-type" content="text/html; charset=utf-8"> <meta name="robots" content="NONE,NOARCHIVE"> 
        <title>`ImportError` at /collection/list</title>

<pre class="exception_value">cannot import name &#x27;`InvalidKeyError`&#x27; from &#x27;jwt.exceptions&#x27; 
(/home/ubuntu/myprojectdir/env-notebook/lib/python3.7/site-packages/`jwt/exceptions.py`)</pre>

#### Solution:
Django is confusing between modules 'jwt' and PyJWT. To avoid this confusion uninstall both modules. Now, install jwt first and then install PyJWT.
