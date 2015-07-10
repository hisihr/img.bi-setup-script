# img.bi-setup-script
Debian img.bi setup script

13 May 2015

#Dotdeb Repo setup

nano /etc/apt/sources.list

deb http://packages.dotdeb.org wheezy all
deb-src http://packages.dotdeb.org wheezy all

wget http://www.dotdeb.org/dotdeb.gpg -O - | apt-key add -

#System update

apt-get update && apt-get upgrade

#System dependencies installation

apt-get install python-pip python-dev build-essential libffi-dev graphicsmagick imagemagick redis-server git-core fontforge spawn-fcgi nginx swig

#Node.js install

cd /tmp 
wget http://nodejs.org/dist/v0.12.2/node-v0.12.2.tar.gz
tar xfz node-v0.12.2.tar.gz
cd node-v0.12.2/
./configure && make && make install

#Node.js Dependencies install

npm install grunt -g
npm install bower -g
npm install grunt-cli -g

#Setup work directoy

mkdir -p /home/img_bi
cd /home/img_bi

#Frontend via git

git clone https://github.com/imgbi/img.bi.git
cd img.bi/

#Frontend configuration

# in config.json setup URLs

nano /home/img_bi/img.bi/config.json
npm install
bower install --allow-root
grunt

#Dependencies API install

easy_install web.py
easy_install M2Crypto
easy_install redis
easy_install bcrypt
easy_install pysha3
easy_install zbase62
easy_install pyutil
easy_install flup

#IMG.bi API via GIT download and upload dir setup

cd /home/img_bi
git clone https://github.com/imgbi/img.bi-api.git
cd /home/img_bi/img.bi-api
mkdir -p /home/img_bi/img.bi-files/thumb

nano /home/img_bi/img.bi-api/code.py

#Das habe ich für meinen Fall geändert  

upload_dir = '/home/img_bi/img.bi-files'
#    Zeile 78 ändern sonst gibt es Fehler beim löschen
##     Die Zeile löschen 
if bcrypt.hashpw(data.password, hashed) == hashed:
##     Und durch die Zeile ersetzen
if bcrypt.hashpw(data.password.encode('utf-8'), hashed) == hashed:

#API testing

chmod +x code.py
chmod +x expired.py
python code.py

WSGIServer: missing FastCGI param REQUESTMETHOD required by WSGI!
WSGIServer: missing FastCGI param SERVERNAME required by WSGI!
WSGIServer: missing FastCGI param SERVERPORT required by WSGI!
WSGIServer: missing FastCGI param SERVERPROTOCOL required by WSGI!
Status: 404 Not Found
Content-Type: text/html

Das Ergebnis sollte so aussehen
API starten und Seite via NGINX bereitstellen

spawn-fcgi -f /home/img_bi/img.bi-api/code.py -a 127.0.0.1 -p 1255

nano /etc/nginx/sites-enabled/vHost.re

server {
    listen                  443 ssl spdy;
    server_name             servername.xxx www.servername.xxx;
    access_log              /dev/null;
    index                   index.php index.html index.htm;
    client_max_body_size    10M;
    location / {
            alias /home/img_bi/img.bi/build/;
    }
    location /api/ {
            fastcgi_param REQUEST_METHOD $request_method;
            fastcgi_param QUERY_STRING $query_string;
            fastcgi_param CONTENT_TYPE $content_type;
            fastcgi_param CONTENT_LENGTH $content_length;
            fastcgi_param GATEWAY_INTERFACE CGI/1.1;
            fastcgi_param SERVER_SOFTWARE nginx/$nginx_version;
            fastcgi_param REMOTE_ADDR $remote_addr;
            fastcgi_param REMOTE_PORT $remote_port;
            fastcgi_param SERVER_ADDR $server_addr;
            fastcgi_param SERVER_PORT $server_port;
            fastcgi_param SERVER_NAME $server_name;
            fastcgi_param SERVER_PROTOCOL $server_protocol;
            fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_script_name;
            fastcgi_pass 127.0.0.1:1255;
    }
    location /download/ {
            alias /home/img_bi/img.bi-files/;
    }

    ssl                             on;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver_timeout 5s;


    ssl_certificate /PATH/TO/SSL/CERT.crt;
    ssl_certificate_key /PATH/TO/SSL/KEY.key;

}

Crontab anlegen zum löschen der Bilder

nano /etc/crontab

* */2 * * * root /home/img_bi/img.bi-api/expired.py

#Finished setup
