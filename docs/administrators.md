Updates to the software
-------

To log into the server, use SSH with your username and password.

```console
ssh username@singlepayercost.org
```

Once inside the server, the project's root directory can be found at /home/singlepayer

```console
cd /home/singlepayer
```

To update the codebase from GitHub, stash and pull.

```console
git stash
git pull
```


Installing software
------

From a fresh Ubuntu 14.0, follow these commands to create and run server:


### Install python 

```console
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.6
```

### Download pip 

```console
mkdir ~/Downloads/
cd ~/Downloads
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```

### Install pip and flask

```console
sudo python3.6 get-pip.py
sudo pip install Flask requests Flask-RESTful termcolor
```

### Cloning repo

```console
cd /home
sudo git clone https://github.com/alexgoodell/single-payer.git
```

### Change permissions

```console
sudo chmod -R 777 single-payer
```

### Start test server 
This is only for testing - not for production

```console
!!!! cd /home/single-payer
!!!! sudo python3.6 index.py
```


### Install Nginx server

```console
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install nginx
```

### Remove Nginx default config

```console
sudo rm /etc/nginx/sites-enabled/default 
```

### Create new nginx config

```console
sudo mkdir -pv /var/www/single-payer
sudo chmod -R 777 /var/www/single-payer
sudo touch /var/www/single-payer/single-payer_nginx.conf
sudo /etc/init.d/nginx start
```

Now edit that file by calling

```console
sudo vi /var/www/single-payer/single-payer_nginx.conf
```

and paste these contents

```
server {
    listen      80;
    server_name localhost;
    charset     utf-8;
    client_max_body_size 75M;

    location / { try_files $uri @yourapplication; }
    location @yourapplication {
        include uwsgi_params;
        uwsgi_pass unix:/var/www/single-payer/single-payer_uwsgi.sock;
    }    
}
```

Create symlink & restart nginx

```console
sudo ln -s /var/www/single-payer/single-payer_nginx.conf /etc/nginx/conf.d/
sudo /etc/init.d/nginx restart
```

At this point, the site should report a **bad gateway**

### Install uWSGI for python 3.6

```console
sudo apt install python3.6-dev uwsgi uwsgi-src uuid-dev libcap-dev libpcre3-dev libssl-dev
cd ~
export PYTHON=python3.6 uwsgi --build-plugin "/usr/src/uwsgi/plugins/python python36"
sudo mv python36_plugin.so /usr/lib/uwsgi/plugins/python36_plugin.so
sudo chmod 644 /usr/lib/uwsgi/plugins/python36_plugin.so
```

### Create uWSGI config file

```console
sudo touch /var/www/single-payer/single-payer_uwsgi.ini
```

Now edit that file by calling

```console
sudo vi /var/www/single-payer/single-payer_uwsgi.ini
```

And paste these contents:

```
[uwsgi]

plugins-dir = /usr/lib/uwsgi/plugins
plugin = python36

#application's base folder
base = /home/single-payer

#python module to import
app = app
module = %(app)

pythonpath = %(base)
pythonpath = /usr/local/lib/python3.6/dist-packages

#socket file's location
socket = /var/www/single-payer/%n.sock

#permissions for the socket file
chmod-socket    = 666

#the variable that holds a flask application inside the module imported at line #6
callable = app

#location of log files
logto = /var/log/uwsgi/%n.log
```

Create the log file directory with appropriate permissions

```console
sudo mkdir -p /var/log/uwsgi
sudo chown -R ubuntu:ubuntu /var/log/uwsgi
```

### Run uWSGI

```console
uwsgi --ini /var/www/single-payer/single-payer_uwsgi.ini
```

### Create emperor

```console
sudo touch /etc/init/uwsgi.conf
sudo vi /etc/init/uwsgi.conf
```

paste in

```
description "uWSGI"
start on runlevel [2345]
stop on runlevel [06]
respawn

env UWSGI=/usr/bin/uwsgi
env LOGTO=/var/log/uwsgi/emperor.log

exec $UWSGI --master --emperor /etc/uwsgi/vassals --die-on-term --uid www-data --gid www-data --logto $LOGTO
```

then:

```console
sudo mkdir /etc/uwsgi
sudo mkdir /etc/uwsgi/vassals
sudo ln -s /var/www/single-payer/single-payer_uwsgi.ini /etc/uwsgi/vassals
sudo chown -R 777 /var/www/
```

Lastly,

```console
sudo start uwsgi <- not currently working
```


