# Setup Nginx with Let’s Encrypt on Ubuntu 22.04

✅ Certbot recommends using their snap package for installation. Snap packages work on nearly all Linux distributions, 
but they require that you’ve installed snapd first in order to manage snap packages. 
Ubuntu 22.04 comes with support for snaps out of the box, so you can start by making sure your snapd core is up to date:
```
sudo snap install core; sudo snap refresh core
```
✅ If you’re working on a server that previously had an older version of certbot installed, you should remove it before going any further:

```
sudo apt remove certbot
```

✅ After that, you can install the certbot package:

```
sudo snap install --classic certbot
```

✅ Finally, you can link the certbot command from the snap install directory to your path, so you’ll be able to run it 
by just typing certbot. This isn’t necessary with all packages, 
but snaps tend to be less intrusive by default, so they don’t conflict with any other system packages by accident:

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

✅ To check, open the configuration file for your domain using nano or your favorite text editor:

```
sudo nano /etc/nginx/sites-available/default
```

✅ And change the `server_name` as necessary (e.g. `notebook-beta.inteam.jp`)

```
server {
    listen 80;
    server_name notebook-beta.inteam.jp;
    .... othe code
}
```
✅ Allowing ufw
```
sudo ufw allow 'Nginx Full'
```

✅ Certbot provides a variety of ways to obtain SSL certificates through plugins. 
The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary. To use this plugin, type the following:

```
sudo certbot --nginx -d notebook-beta.inteam.jp
```

✅ Restart nginx
```
sudo systemctl restart nginx
```

### Verifying Certbot Auto-Renewal
✅ Let’s Encrypt’s certificates are only valid for ninety days. 
This is to encourage users to automate their certificate renewal process. 
The certbot package we installed takes care of this for us by adding a systemd timer 
that will run twice a day and automatically renew any certificate that’s within thirty days of expiration.

✅ You can query the status of the timer with systemctl:
```
sudo systemctl status snap.certbot.renew.service
```

✅ To test the renewal process, you can do a dry run with certbot:
```
sudo certbot renew --dry-run
```


### 🔥 Can not log into SSH  related issues !<hr>
If firewall `(ufw)` is `enabled` by any chance, it may raise this issue of not letting you log into `ssh`.
I was using `mobaxterm`and keep on failing to login.

What i did:
User Fabián Bertetto on stackoverflow
https://stackoverflow.com/questions/41929267/locked-myself-out-of-ssh-with-ufw-in-ec2-aws







