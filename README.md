# Formation-Deployement-With-Azure

# Node.js Deployment

> Steps to deploy a Node.js app to Azure using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt

## 1. Sign up for Azure portal

## 2. Create a new virtual machine and log in via ssh
Exple : Open your cmd and type : ssh Your_Machine_Name@Public-IP-address


## 3. Install Node/NPM
```
sudo apt-get update && sudo apt upgrade -y
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -

sudo apt-get install -y nodejs

node --version
```

## 4. Clone your project from Github
There are a few ways to get your files on to the server, I would suggest using Git
```
git clone yourproject.git
```

### 5. Install dependencies and test app
```
cd yourproject
npm install
npm start (or whatever your start command)
# stop app
ctrl+C
```
## 6. Setup PM2 process manager to keep your app running
```
sudo npm i pm2 -g
pm2 start app (or whatever your file name)

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```
Link For more information about PM2: https://pm2.keymetrics.io

### You should now be able to access your app using your IP and port. Now we want to setup a firewall blocking that port and setup NGINX as a reverse proxy so we can access it directly using port 80 (http)

## 7. Setup ufw firewall
```
sudo ufw enable
sudo ufw status
sudo ufw allow ssh (Port 22)
sudo ufw allow http (Port 80)
sudo ufw allow https (Port 443)
```

## 8. Install NGINX and configure
```
sudo apt-get update && sudo apt-get install nginx -y
```
### To verify that ngnix is active (running)
```
sudo systemctl status ngnix  
```
### Create New Ngnix File Config 
```
cd /etc/nginx/sites-available/
```
### Change "yourNewConfigFileName" with your prefer Name
```
sudo nono yourNewConfigFileName
```
Add the following to new file 
### * Make Sure to change the root path with your build folder related to your Application
### * Same thing for "server_name" with your public IP_Adresss of your Azure Instance(VM)
```
server {
        listen 80;
        listen [::]:80;

        root /home/ubuntu/my-app/build;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name 172.206.253.48;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
#               add_header 'Access-Control-Allow-Origin: $http_origin';
#               add_header 'Access-Control-Allow-Origin: GET, POST, DELETE, PUT, PATCH, OPTIONS';

        }

}
```
# Check NGINX config
```
sudo nginx -t
```
# Restart NGINX
```
sudo service nginx restart
```
* Return to the your root diacertory ~/
```
cd
```
### * Now we need to create the index.html of our front-App so enter to your application Folder and run :
```
npm run build       =====>(after runinng this command line a new folder created titled build)
```


***********************************************************************************************
### * !!!! This is another config for anthoer purpose don't execute it !!!  
```
Add the following to the location part of the server block


    server_name "your-Public-IP-Adress"  or  "yourdomain.com"  or  "www.yourdomain.com" ;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
                proxy_pass http://localhost:3000; #whatever port your app runs on
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }

```
*************************************************************************************************


### You should now be able to visit your IP with no port (port 80) and see your app. Now let's add a domain

## 9. Add domain in Azure
In Azure, go to All Resources, and choose your machine resource of your virtual machine (you will find something like this: "Name_Of_Your_ResourceMachine-ip")
Click on it on go to the setting on the top left then Configuation and simply add your "DNS name label" you want .

PS : It may take a bit to propogate

10. Add SSL with LetsEncrypt
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run
```

Now visit https://yourdomain.com and you should see your Node app
