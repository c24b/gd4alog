# INSTALL

Pour le moment l'installation a été prévue simplement sur le système d'exploitation Ubuntu 20.04

clone this repository

git clone **_<changeme>_**

### System requirements

sudo apt-get update
sudo apt install apt-transport-https ca-certificates wget
sudo apt-get install -Y git python3 virtualenv
sudo apt-get install python3-pip python3-dev nginx

#### MongoDB

```
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list

sudo apt update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
sudo systemctl status mongod
sudo systemctl enable mongod
```

#### ElasticSearch
```
sudo apt install openjdk-8-jre-headless
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elasticsearch-6.list
sudo apt update
sudo apt install elasticsearch
sudo systemctl enable --now elasticsearch.service
curl -X GET "localhost:9200/"
```

* Troubleshooting:
  If fails to connect to port 
  ```bash
  sudo chown -r $USER /var/lib/elasticsearch/
  sudo chown -r $USER /etc/elasticsearch/
  ```
  
#### ArgosTranslate
```
sudo snap install argos-translate
argospm install translate-fr_en
argospm install translate-en_fr
```

### BACK

```
cd back
virtualenv .env
source .venv/bin/activate
pip install -r requirement.txt
source .env
```
#### Initialize database

```
source gd4h/back/.venv/bin/activate
cd back/scripts/
python script/init_db.py
```

#### Indexing content

```
source back/.venv/bin/activate
cd scripts/
python create_index.py
```

### FRONT

```

cd front/flask_app
virtualenv .env
source .venv/bin/activate
pip install -r requirement.txt
```


# Deployement

### BACK

* nginx 

/etc/nginx/sites-available/gd4h-api

```

server{
       server_name api.gd4h api.gd4h.fr;
       location / {
           include proxy_params;
           proxy_pass http://127.0.0.1:3000;
       }
}
```
sudo ln -s /etc/nginx/sites-available/gd4h-api /etc/nginx/sites-enabled/


sudo systemctl restart nginx.service


* Service systemd
  
Edit /etc/systemd/system/gd4h-api.service

```
[Unit]
Description=Gunicorn instance to serve gd4h-api
After=network.target

[Service]
User=gd4h-admin
Group=www-data
WorkingDirectory=/home/gd4h-admin/GD4H/back/
Environment="PATH=/home/gd4h-admin/GD4H/back/.venv/bin"
ExecStart=/home/gd4h-admin/GD4H/back/.venv/bin/gunicorn --bind=127.0.0.1:3000 -w 4 -k uvicorn.workers.UvicornWorker main:app

[Install]
WantedBy=multi-user.target
```

sudo systemctl start gd4h-api.service


### FRONT


* nginx 

Edit  `/etc/nginx/sites-available/gd4h-front` with sudo privileges

```

server{
       server_name gd4h gd4h.fr;
       location / {
           include proxy_params;
           proxy_pass http://127.0.0.1:5000;
       }
}
```
sudo ln -s /etc/nginx/sites-available/gd4h-front /etc/nginx/sites-enabled/


sudo systemctl restart nginx.service


* Service systemd
  
Edit /etc/systemd/system/gd4h-front.service

```
[Unit]
Description=Gunicorn instance to serve gd4h-front
After=network.target

[Service]
User=gd4h-admin
Group=www-data
WorkingDirectory=/home/gd4h-admin/GD4H/front/flask_app
Environment="PATH=/home/gd4h-admin/GD4H/front/flask_app/.venv/bin"
ExecStart=/home/gd4h-admin/GD4H/front/flask_app/.venv/bin/gunicorn --bind=127.0.0.1:5000 -w 4 -k uvicorn.workers.UvicornWorker main:app

[Install]
WantedBy=multi-user.target
```

`sudo systemctl start gd4h-front.service`

