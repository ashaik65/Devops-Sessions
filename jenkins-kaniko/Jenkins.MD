### How to deploy Jenkins with Our custom domain and ssl ###

For install jenkins we need to follow below shell script i am using ubuntu 20.04 this script also work for ubuntu 22.04

vi jenkins.sh
```yaml
#! /bin/bash

sudo apt update
sudo apt install openjdk-11-jre -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt install -y maven
sudo apt-get install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

```
Run script using this command

sh jenkins.sh


After that we need to check using jenkins using ec2 instance id for example:- 34.203.34.186:8080  for checking purpose but our end goal its should not be run on this after configuring nginx reverse proxy will be remove port 8080 from ec2 instnace security group and check

### step-by-step guide to configure SSL on Jenkins using Let's Encrypt and NGINX reverse proxy ###

1. Install NGINX on your server if it's not already installed. You can do this by running the following command:
```yaml
sudo apt-get update

sudo apt-get install nginx

2. Create a new server block configuration file for Jenkins. You can do this by creating a new file in the /etc/nginx/sites-available/ directory. For example:

vi /etc/nginx/sites-available/jenkins

3. Add the following content to the file:

server {
    listen 80;
    server_name jenkins.example.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;        
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

Note: Replace jenkins.example.com with your Jenkins domain name.
```

4. Create a symbolic link to the server block configuration file in the /etc/nginx/sites-enabled/ directory:
```yaml

sudo ln -s /etc/nginx/sites-available/jenkins /etc/nginx/sites-enabled/

5. Test the NGINX configuration and restart the NGINX service:

sudo nginx -t
sudo systemctl restart nginx

6. Install Certbot, the Let's Encrypt client, by running the following commands:

sudo apt-get update

sudo apt-get install certbot python3-certbot-nginx

7. Obtain an SSL certificate for your Jenkins domain name using Certbot:

sudo certbot --nginx -d jenkins.example.com

Note: Replace jenkins.example.com with your Jenkins domain name.

Follow the prompts to provide your email address and agree to the Let's Encrypt terms of service.

Choose whether or not to redirect all HTTP traffic to HTTPS. This is recommended for security purposes.

Test your SSL configuration by visiting your Jenkins domain name using HTTPS.

That's it! You have successfully configured SSL on Jenkins using Let's Encrypt and NGINX reverse proxy.

### for jenkins run behind vpn need to configure like this ###
vi /etc/nginx/sites-available/jenkins

server {
    server_name jenkinsdev.techmobi.site;

    location / {
        allow 54.226.177.182; #allow pritunl ip here
        deny all; # add this property as well
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;        
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/jenkinsdev.techmobi.site/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/jenkinsdev.techmobi.site/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = jenkinsdev.techmobi.site) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
  

    listen 80;
    server_name jenkinsdev.techmobi.site;
    return 404; # managed by Certbot

}


