Below are the requires for this project. 

- Disable ssh login for root
- Can run sudo commands as grader
- create ssh key for grader
- extract private key for grader
- Change ssh port to run on 2200
- Allow no ports by default
- Allow ssh connections on 2200 through firewall
- Allow web connections on 80 through firewall
- Allow NTP connections on 123
- Key based SSH authenication is enforced
- Update all system packages (using apt update && apt upgrade)
- web server responds to port 80
- Database server has been configured to serve data
- web serve has been config for wsgi

# The beginning:

1. Create Digitial account the cost is 5 dollars for 1 gb(it is plenty for this type of application.
* On the left hand side select Droplet and create a ubuntu 18.04 server.
* **When creating a digital ocean droplet it will ask for ssh key**

2.On your local machine bring up a terminal or shell and read the below carefully

  * **type ssh-keygen this will prompt a question "enter file in which to save the key" example below Enter file in which to save the key** _(/home/coloed3/.ssh/id_rsa): (home/coloed3/.ssh/project)
  
  * Add the above text in parentheses with your folder location and name of the file you want i used project.
  
   * Once that is created to get the key type the following command cat ~/.,ssh/project.pub copy the ssh key it will look like this
   
   __ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC/U18ayQkLgSBtTHIROM3xWJW9hvq7P28jJUtjd7awhUPeY6UrLEsAIaXCF+iepwmGQsbqirfh6lHB2Tja3GLZZQUPS11JYDRxQeUs3E7QL+rQwOBwWbqwQ55ewmXUXApqoR5uWbdakMAp8qjnQtzz4amrHztZYdAtV210XQMrqyoHkcK3CghGGMXGqvEwCjGDrm8ATWn++QDdYgOhNpAgP3yleti8Yo1ty9JBm4gcvg7eSjBCJ33c50sAHmnWz7277AKOX+rs/Cp4cC6kStr96dB5NM4xvd/Tdlaw0BrFcikYyBjzv1CT6tFDFJiqKLYfXt/Pw55aKr51OWqaWOp/ coloed3@coloed3-desktop into the digital ocean ssh box__
   * Change the digital ocean password- using the password they provided(in your email )
   
   #Create sudo user by using command adduser name / and create password use below as a reference
   https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
   1. adduser grader set and confirm grader password we used grader as password.
   * usermod -aG sudo grader
   * su - grader
   * once this is complete verify user has root access by running sudo apt update && sudo apt upgrade
   ##Adding ssh access to the user grader password is grader
   1. ssh into the digtial droplet server by using 
   1.  You will see grader@ubunut-s-1vcpu-1gb-sfo2-01
   1. mkdir .ssh
   1. chmod 700 .ssh
   1. create a authorized_keys by typing touch authorized_keys
   1. Chmod 644 authorized_keys
   1. __copy ssh and input that ino tthe authorize_keys using nano authorize_keys__
   __for a full walk through on how ot create your key and use it https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804__
   ## Disable root 
   1. Ssh in with grader@xxxxx with password above once login in
   * sudo nano /etc/ssh/sshd_config and change the following permission from yes to No Permitrootlogin once completed type nano service sshd restart -
   * Test login with root user on different shell to verify that the access to root is denied
   ## Create the apache2 server on digitalocean and modify UFW(fallwall)
   * __Reference: https://help.ubuntu.com/lts/serverguide/firewall.html.en__
   * sudo apt update
   * sudo apt install apache2  
   * adjust the firewall to allow web traffic by doing the following command sudo ufw app list
   * sudo ufw app list
   * sudo ufw app info "Apache full"
   * will show 80,443
   * sudo ufw allow proto tcp from any to any port 80 allows 80 to be used through the fire wall
   * sudo ufw enable ( will enable firewall
   * sudo ufw status to verify port 80

## Verification step for server
1.Install PHP (we will use to verify server by using <?php phpinfo();?> cd /var/www/html/ - nano index.html and input the command
* sudo apt install php libapache2-mod-php php-mysql
* Restart apache server sudo systemctl restart apache2
* sudo systemctl status apache2
# Change ssh port to run on 2200
1. change ssh port to run on 2200
2. nano /etc/ssh/sshd_config and change port 2200 ctrl +o / ctrl+x
3. service sshd restart
4. Add port 2200 to ufw firewall so it allows to ssh in using command ufw allow proto tcp from any to any port 2200
5. login with ssh name@134.209.55.142 -p 2200
## Deny all other connections
* We did sudo ufw default deny incoming
1. __We tested this by using the hello world example for node js. Doing a curl command locally to port 3000 worked. We could not access that port remotely.  __
## Allow connections on ntp 123
1. sudo ufw allow 123
2. update && upgrade all packages
3. sudo apt install update && upgrade

# Setting up flaskapp 
1. Clone https://github.com/coloed3/Item-catalog-udacity.git into /var/www/python
* __If you do not have git installed please follow this link https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)__ to install on what ever os you are on. Git is pre-installed on linux
* Sudo rm catalogsdatabase.db to delete database in repo
## Installing the right packages:
* sudo apt-get install python3 to install python
* sudo apt install python3-pip to install pip3

* When pip3 is done install the following packages
* install the following packages with pip3 install flask, flask-sqlalcahemy, oauth2client

# Setting up postgres 
1. Install postgres if using digital ocean on the “create droplet page” select postgres, this will create the following file /etc/apt/sources.list.d/pgdg.list
* Nano into  /etc/apt/sources.list.d/pgdg.list and paste the following deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main 
2. Once that is completed sudo apt install postgressql
3. When postgres is downloaded do the following command sudo su - postgres and type psql to get into the shell.
 * Then do the following in order:
 * Create database catalog; 
 * create user catalog 
 * alter role name created with password “give password” 
 * then we will grant the privileges by Grant all privileges on DATABASE catalog to username;
 

# Creating the wsgi and service that will run Our project
1. sudo nano /etc/systemd/system/ItemCatalog.service  and paste the below code inside and  x + ctrl to exit and save 
[Unit]
Description=Item Catalog Udacity Service
After=network.target
After=systemd-user-sessions.service
After=network-online.target

[Service]
User=grader
ExecStart=/var/python/Item-catalog-udacity/start.sh
Restart=on-failure
RestartSec=30
RuntimeDirectoryMode=777

[Install]
WantedBy=multi-user.target
## Create an Sh file
* Nano start.sh in the python app directory and paste the following
"#!/bin/bash"
* cd /var/python/Item-catalog-udacity && python3 app.py
* In the python app directory sudo chmod 777 -R . to give access to the service file
* To start the service sudo systemctl start ItemCatalog.service
* Enable the service incase of server reboot with sudo systemctl enable ItemCatalog.service
* Create apache2 proxy reversal
* sudo a2enmod proxy proxy_http
* sudo service apache2 restart
* sudo nano /etc/apache2/sites-available/000-default.conf
* In the file do the following comment out document root
> ProxyPreserveHost On
 ProxyPass / http://127.0.0.1:5000/
ProxyPassReverse / http://127.0.0.1:5000/ 4. sudo service apache2 restart to restart apache2 and test server




##Server will be up and running, per the requirements postgres was recommended but i choose to stay with sqlite.

# References:
* https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804
* https://www.digitalocean.com/community/questions/add-ssh-key-after-creating-a-droplet
* https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
* https://help.ubuntu.com/lts/serverguide/firewall.html.en\
* https://askubuntu.com/questions/919054/how-do-i-run-a-single-command-at-startup-using-systemd
* https://stackoverflow.com/questions/30642894/getting-flask-to-use-python3-apache-mod-wsgi
