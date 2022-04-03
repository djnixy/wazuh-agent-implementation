# wazuh-agent-implementation

### DVWA and Wazuh Server

#### Installation

You will need to provision 2 VM (for example, I personally use AWS):
 
ALPHA VM for DVWA, mariaDB and Wazuh agent:
t3.small (2 vCPU, 2GB RAM)
 
BETA VM for Wazuh Server:
t3.medium (2 vCPU, 4GB RAM)

- - -

#### BETA VM Installation
 
Run this script to install Wazuh server:
```
apt update && apt upgrade
curl -so ~/unattended-installation.sh https://packages.wazuh.com/resources/4.2/open-distro/unattended-installation/unattended-installation.sh && bash ~/unattended-installation.sh
```
The installer will inform you which credentials you can use to login to your Wazuh server.

- - -

#### ALPHA VM Installation 
Run the scripts below on ALPHA VM as root:
```
apt-get update && apt-get -y install apache2 php php-mysqli php-gd libapache2-mod-php ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 
apt-get update
apt-get -y install docker-ce docker-ce-cli containerd.io
```

****Run mariaDB docker container:****
```
docker run -itd --rm --name dvwadb -e MARIADB_DATABASE=dvwa -e MARIADB_USER=dvwa -e MARIADB_PASSWORD=p@ssw0rd -e MARIADB_ROOT_PASSWORD=p@ssw0rd -v "dvwadb-data:/var/lib/mysql" -p 3306:3306 mariadb:latest
```
****Deploy app to apache web server:****
```
git clone https://github.com/djnixy/DVWA
rm -r /var/www/html/index.html
rsync -avP DVWA/ /var/www/html/
```
****Install Wazuh Agent****
 
Please note that you need to specify BETA VM’s IP address when installing Wazuh agent.

Run the scripts below as root:
```
curl -so wazuh-agent-4.2.6.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.2.6-1_amd64.deb && sudo WAZUH_MANAGER='<your wazuh server ip>' WAZUH_AGENT_GROUP='default' dpkg -i ./wazuh-agent-4.2.6.deb
 
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent
```
Now you can access DVWA using default credentials:
```
User: admin
Password: password
```
![image](https://user-images.githubusercontent.com/17786996/161450639-8296cd16-9bb8-4bc1-835b-213eeda15ad0.png)


Click Create / Reset Database to initialize database.

- - -

### DEMO

Cross Site Scripting attack:
 
![image](https://user-images.githubusercontent.com/17786996/161450644-d1935ef0-74da-4be9-926a-410facc22491.png)
![image](https://user-images.githubusercontent.com/17786996/161450646-9d7d4fa3-135d-4504-bd1d-61f200df59fd.png)
![image](https://user-images.githubusercontent.com/17786996/161450648-8fbab96d-e01e-4f60-bcaf-16647d279e81.png)


 
 
Sql Injection attack:
![image](https://user-images.githubusercontent.com/17786996/161450652-cc0638ca-8805-4587-9bc9-55bed8c0e24f.png)
![image](https://user-images.githubusercontent.com/17786996/161450658-b09d597c-b8a3-4085-bb0c-d9089518291b.png)


 
After some attacks, let’s check the logs on Wazuh Server. You should be able to see your installed agents on Wazuh’s dashboard:
![image](https://user-images.githubusercontent.com/17786996/161450663-f3533eac-c869-4e78-a0e5-d19b2baed32b.png)

Click the Active agent’s number and you will see this screen:
![image](https://user-images.githubusercontent.com/17786996/161450400-bb38a11d-6d78-4a86-b281-b34a82603950.png)

Click it and go to Security events:
 
![image](https://user-images.githubusercontent.com/17786996/161450388-20fb7f84-20f7-4d77-8da7-2f2b4732a0d1.png)



 
Click on Events to check the log from Wazuh agent to Wazuh server:
![image](https://user-images.githubusercontent.com/17786996/161450379-e2074e45-10fe-49d3-839d-0e0deed15913.png)
![image](https://user-images.githubusercontent.com/17786996/161450381-66b66d59-6991-4444-ab90-3441f587e2f1.png)

![image](https://user-images.githubusercontent.com/17786996/161450367-f19475f6-cdb3-4f0a-9c69-3094dee77727.png)


 
