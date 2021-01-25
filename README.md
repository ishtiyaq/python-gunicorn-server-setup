# Python - Gunicorn Server Setup

```bash
sudo apt update
sudo apt upgrade

adduser ishtiyaq
usermod -aG sudo ishtiyaq

sudo apt install python python3-pip
```

```bash
git clone https://github.com/egorsmkv/simple-django-login-and-register.git mysite
cd mysite
pip3 install pipenv

vi ~/.bashrc
PYTHON_BIN_PATH="$(python3 -m site --user-base)/bin"
PATH="$PATH:$PYTHON_BIN_PATH"

pipenv install
pipenv shell
pipenv install django
pipenv install gunicorn

python source/manage.py migrate
python source/manage.py collectstatic --no-input

cd source
gunicorn --bind 0.0.0.0:8000 app.wsgi:application
```

```bash
vi app/conf/development/settings.py

allowed_host=['*']
```

## Nginx server setup

```bash
sudo apt install nginx
sudo vi /etc/nginx/sites-enabled/demo.conf
```

```bash
server{
    listen 80;
    server_name 128.199.24.37;

    location / {
            include proxy_params;
            proxy_pass http://unix:/home/ishtiyaq/mysite/source/app.sock;
    }

    location /static/ {
            autoindex on;
            alias /home/ishtiyaq/mysite/source/content/static/;
    }
}
```

```bash
sudo nginx -t
sudo systemctl restart nginx
```

```bash
gunicorn --workers 3 --bind unix:/home/ishtiyaq/mysite/source/app.sock app.wsgi:application
```

## Gunicorn daemon setup

```bash
sudo apt install supervisor
```

### Get guniocorn path

```bash
which gunicorn

# Output:
/home/ishtiyaq/.local/share/virtualenvs/mysite-dJ5dHllL/bin/gunicorn
```

### Create supervisor conf file for gunicorn

```bash
vi /etc/supervisor/conf.d/gunicorn.conf
```

```bash
[program:gunicorn]
directory=/home/ishtiyaq/mysite/source/
command=/home/ishtiyaq/.local/share/virtualenvs/mysite-dJ5dHllL/bin/gunicorn --workers 3 --bind unix:/home/ishtiyaq/mysite/source/app.sock app.wsgi:application
autostart=true
autorestart=true
stderr_logfile=/var/log/gunicorn/gunicorn.err.log
strout_logfile=/var/log/gunicorn/gunicorn.out.log

[group:guni]
programs=gunicorn
```

```bash
sudo mkdir /var/log/gunicorn

sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl restart all
sudo supervisorctl status
```


## Debugging

```bash
tail -f /var/log/gunicorn/gunicorn.err.log
tail -f /var/log/nginx/error.log
```
