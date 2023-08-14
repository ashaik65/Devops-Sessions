### Pritunl setup on ubuntu 20.04 ###

1. Add the Pritunl repository to your sources list:

echo "deb http://repo.pritunl.com/stable/apt focal main" | sudo tee /etc/apt/sources.list.d/pritunl.list

2. Add the Pritunl GPG key:
sudo apt-get install dirmngr gnupg -y

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 7568D9BB55FF9E5287D586017AE645C0CF8E292A

3. Update the APT package cache:
sudo apt-get update

4. Install MongoDB:

sudo apt-get install mongodb-server -y

5. Start the MongoDB service:

sudo systemctl start mongodb

6. Enable MongoDB to start on boot:

sudo systemctl enable mongodb

7. Install Pritunl server:

sudo apt-get install pritunl -y

8. Start the Pritunl service:

sudo systemctl start pritunl

9. Enable Pritunl to start on boot:

sudo systemctl enable pritunl


To Access use https://your-server-ip-address. in this case our ec2 address and after that while setup it will ask domain name so provide domain name and add A record entry in Route53


### Now we need to whitelist jenkins domain name with pritunl VPN so outsider user not be access our jenkins website only the internal users who have access of VPN those will be access ###







