# Install
Setup and launch of the eStore - Hermes

The instruction is relevant only for the latest version(for free) which is on the site https://bothermes.com  

# The short version is for hackers:  
This is a standard ASP.NET Core app.  

# For everyone else a detailed installation example on Debian 9:  

You will not fit hosting you need at least a vps server.  
You will also need a domain name that points to your server.

To start install .NET Core 2.2 Runtime following the instructions from the official website  
https://dotnet.microsoft.com/download/linux-package-manager/debian9/runtime-2.2.7  


For example I will use free certificates for my site and it will have a false name example.com  
# FREE SSL  

apt-get install certbot  
service nginx stop  
service apache2 stop  
certbot certonly --standalone -d example.com  


# NGINX  
In the example I use Nginx and so Apache will remove  

service apache2 stop  
apt autoremove apache2*  
apt autoremove nginx*  
apt install nginx-full nano  







# The configuration for the site
nano /etc/nginx/sites-available/srv0.conf
```
server {
    listen 80;
    server_name example.com;
    access_log /var/log/nginx/srv0.log;
    error_log /var/log/nginx/srv0_error.log;
    rewrite ^(.*) https://example.com$1 permanent;
    server_tokens off;
}
server
{
    listen 8443 ssl;
    ssl on;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
	ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

    ssl_session_timeout 5m;

    ssl_stapling on;
	ssl_stapling_verify on;
	resolver 127.0.0.1 8.8.8.8;
    server_name example.com;
    root /var/www/html;
    access_log /var/log/nginx/srv0.log;
    error_log /var/log/nginx/srv0_error.log;
    rewrite_log on;
    server_tokens off;
    location / {
        proxy_pass         http://localhost:5000; #For VPN http://10.8.0.2:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
     }
}
```







# Configuration for the server
rm /etc/nginx/nginx.conf  
nano /etc/nginx/nginx.conf  
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
events {
	worker_connections 1024;
	multi_accept on;
}
http {
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	include /etc/nginx/mime.types;
	default_type application/octet-stream;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_prefer_server_ciphers on;
	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;
	gzip on;
	gzip_disable "msie6";
	gzip_vary on;
	gzip_proxied any;
	gzip_comp_level 6;
	gzip_buffers 16 8k;
	gzip_http_version 1.1;
	gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```






# The LINK for the config
ln -s /etc/nginx/sites-available/srv0.conf /etc/nginx/sites-enabled/srv0.cong

# Check the syntax
nginx -t

# Running Nginx
service nginx start  
service nginx restart  
service nginx status  






# CREATING A SERVICE
(paths in small letters) (do not forget to specify the working directory - WorkingDirectory)  
nano /etc/systemd/system/bothermes.service  
```
[Unit]
Description=Example.COM .NET Web API App running on Debian
[Service]
WorkingDirectory=/var/www/html/
ExecStart=/usr/bin/dotnet /var/www/html/Hermes.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=bot-hermes
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
Environment=DOTNET_CLI_TELEMETRY_OPTOUT=false
[Install]
WantedBy=multi-user.target
```



# Service registration

systemctl enable bothermes.service  







# Site update / download  
https://bothermes.com/

service bothermes stop  
rm -rf /var/www/html/*  

Here you copy the application to the server in a convenient way  

Be careful if you update the site you can not delete the database of customers, products and   others.
All databases have an extension."db " example of a command that will not delete the database.  

```
find /var/www/html -type f ! -iname "*.db" -delete  
```
Be sure to set the correct access to the application files  
chown -R www-data:www-data /var/www/html/






#  Edit the settings file "HermesOptions.json"
In this example I will use a non standard port to connect the bot to the telegram API 8443  
nano /var/www/html/HermesOptions.json  
ManagerID && AdminID - I will change later when I look them up in the working bot.  
```
{
  "AllowedHosts": "*",
  "Url": "https://example.com:8443/{0}",
  "Name": "MySuperStore",
  "Key": "KEY-KEY-KEY-KEY-KEY-KEY-KEY-KEY-KEY-KEY",
  "SupportContact": "http://t.me/MySupportContact",
  "ManagerID": "12345678",
  "AdminID": "12345678",
  "EventPurchase": "true",
  "EventCreateOrder": "true",
  "EventCanceledOrder": "true",
  "EventOutOfGoods": "true",
  "EventNewClient": "true",
  "EventError": "true",
  "EventWarnings": "true"
}
```


# Launch and registration of the bot

service bothermes start  
service bothermes status  








# TELEGRAM
Register Webhook on your port, And check how everything works

https://api.telegram.org/bot{ API KEY }/setWebhook?url=https://example.com:8443/api/message/update  
https://api.telegram.org/bot{ API KEY }/getWebhookInfo  


# Add a bot in contact and send the command "/info" to see your ID.  
Change it in the settings and restart the bot.  

service bothermes stop
nano /var/www/html/HermesOptions.json

```
  "ManagerID": "4654547897",
  "AdminID": "4654547897",
```

service bothermes restart  







# LAST STEP

Now you have access to the admin panel  
https://example.com:8443/admin/  

On the page you will see an access token of 6-7 digits.  
They should be sent to the bot command  
```
/auth@123456
```

After the bot reports that access is open you can click the unlock button on the site


# OTHER INFORMATION
There is a list of standard commands that are better
to immediately add to the bot control menu via BotFather  
/setcommands  

```
help - Help
settings - Settings
menu - Menu
info - About me
```

# SQL BROSWER - TOOLS  
If you are wondering what is contained in the DB

https://sqlitebrowser.org/  
https://sqlitebrowser.org/dl/#windows  


# This is the end

It remains to add the addresses of wallets for receiving money and add goods.  
Fill in the section with news and rules for customers.  
And of course invite friends to your store.  

If you are experiencing difficulties with setting up and running your store-we will help you.  
More information on the website https://bothermes.com  
