# Project Details
  Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers

 * Url : ec2-35-176-122-0.eu-west-2.compute.amazonaws.com
 
 * SSH Port : 2200
 
 * IP address : 35.176.122.0
 
#  configuration 
   * Create account on Amazon Web Service (AWS) LightSail  and then select os only then chose ubunt 
   
   * use  git bash or other terminal and follow this steps 
     
	 - Download the instance's private key by navigating to the Amazon Lightsail 'Account page'.
	 
	 - A file with extension .pem will be downloaded; open it.
   	 
	 
	 - Copy the text and put it in a file called lightkey.rsa (or any other name to help you could recall 	   anytime) in the local machine ~/.ssh/ director
	 
	 - Run chmod 600 ~/.ssh/lightkey.rsa
	 
	 - Run ssh -i ~/.ssh/lightkey.rsa ubuntu@your-public-ip-address, to login to the VM.
 

# Update and Upgrade existing packages

   - sudo apt-get update
   
   - sudo apt-get upgrade
   
   
# Security 
   
   we will  Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) and deny port 22 by follow this steps 
   
   - sude ufw status to check if firewall is active or inactive 
   
   - changing the SSH port from 22 to 2200 (open up the /etc/ssh/sshd_config file sudo nano    /etc/ssh/sshd_config, change the port number on line 5 to 2200, then restart SSH by running
     sudo service ssh restart 
	 
   - sudo ufw default deny incoming   to block any thing 
   
   - sudo ufw default allow outgoing  to allow any thing 
   
   - sudo ufw allow ssh  			  to allow SSH
   
   - sudo ufw allow 2200/tcp  		  to allow port (2200/tcp)
   
   - sudo ufw www                     to allow www HTTP (Port 80)
   
   - sudo ufw allow 123/udp           to allow NTP (Port 123)
   
   - sudo ufw deny 22                 to deny port 22 we change it to port 2200
   
   - sudo ufw enable                  to enable firwall 
   
   - sudo ufw status                  to check the allowed and denied port  
   
   - then we need to update the firewall configuration on the lightsail AWS instance and change the same ports as configured above by chose manage the instance and then go to the networking 
   
   - to log in again to vm  
   ssh -i ~/.ssh/lightkey.rsa -p 2200 ubuntu@your-public-ip-address 
   
   
# adding new users 
  we will  add user grader and give him  sudo permissions and  access to virtual machine 
   - sudo adduser grader
   
   -  Enter the password
   
   -  su- grader         to switch and enter pass 
   
   -  exit
   
   -  sudo visudo  
   
   -  add the following line after to the ones that look same :grader ALL=(ALL:ALL) 
   
   -  Save and close the visudo file
 
   to give grader access to vm follow this steps 
 
   - Open another terminal  on your local machine and run
   
      * ssh-keygen 		to make private and public key give the file name (grader_key)
	 
	  * Enter in a passphrase
	 
   -  now Log in to the virtual machine 
   
   -  Switch to grader's home directory(~), and create a new .ssh directory by  mkdir .ssh
	 
   -  touch .ssh/authorized_keys
   
   -  go back to another terminal that you have opened 
   
   -  cat ~/.ssh/grader_key.pub
   
   -  Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine
   
   -  chmod 700 .ssh                     on the vm 
   
   -  chmod 644 .ssh/authorized_keys     on the vm
   
   -  chown -R grader:grader .ssh        on the vm
   
   -  Make sure key-based authentication is forced (log in as grader, open the /etc/ssh/sshd_config file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file; 
   
   - sudo service ssh restart
   
   - ssh -i ~/.ssh/grader_key -p 2200 grader@your-public-ip-address
   
   
# Deploying Item Catalog 

  ## change Time zone 
  
  - sudo timedatectl set-timezone UTC
  
  ## install apache 
  
  - sudo apt-get install apache2      
    
  - sudo apt-get install python-setuptools   to install mode_wsgi
               
  - sudo service apache2 restart
   
  ## install git 
  
  we install git and it is easy for clonning project to vm 
  
  - sudo apt-get install git 
  
  - cd var/www 
  
  - sudo mkdir catalog
  
  - cd catalog 
  
  - sudo git clone https://github.com/MekhailMaged96/item-catalog catalog
  
  ## basic configuration for prepare ubuntu 
  
   
  - cd /var/www/catalog/catalog
  
  - sudo mv main.py __init__.py        to change the name of main.py to __init__.py
  
  - cd /var/www/catalog 
  
  - sudo nano catalog.wsgi 
  
  - insert the following text inside 'catalog.wsgi' and be careful for python indentation to avoid erros:
	 import sys
	 import logging
	 logging.basicConfig(stream=sys.stderr)
	 sys.path.insert(0, "/var/www/catalog/")
	 from catalog import app as application
	 application.secret_key = 'super_secret_key'
  
  
  ## Install virtual environment 
  
   - sudo apt-get install python-pip
   
   - sudo pip install virtualenv
   
   - sudo virtualenv venv    		to Create a new virtual environment 
   
   - source venv/bin/activate 	    to Activate the virutal environment
   
   - sudo chmod -R 777 venv 
   
   
  ## install falsk 
  
   - sudo pip install Flask
   
   - sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
  
  ## enable virtual host 
  
  - sudo nano /etc/apache2/sites-available/catalog.conf  to create file catalog.conf 
  
  - Insert the following text inside 'catalog.conf' :
	  <VirtualHost *:80>
	ServerName your-public-ip-address
	ServerAdmin admin@your-public-ip-address
	ServerAlias hostname
	WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv/lib/python2.7/site-packages
	WSGIProcessGroup catalog      
	WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	<Directory /var/www/catalog/catalog/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/catalog/catalog/static
	<Directory /var/www/catalog/catalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
	  </VirtualHost>

  - nslookup your-public-ip-address 		to know your hostname 

  - sudo a2ensite catalog                   to enable virtual host
  
  ## install  PostgreSQL
   
   - sudo apt-get install libpq-dev python-dev
   - sudo apt-get install postgresql postgresql-contrib
   - sudo su - postgres
   - psql
   - CREATE USER catalog WITH PASSWORD 'password';
   - ALTER USER catalog CREATEDB;
   - CREATE DATABASE catalog WITH OWNER catalog;
   - \c catalog
   - REVOKE ALL ON SCHEMA public FROM public;
   - GRANT ALL ON SCHEMA public TO catalog;
   - \q
   - exit
   - Change create engine line in your __init__.py and database_setup.py to: engine = create_engine('postgresql://catalog:password@localhost/catalog')
		Then run the database setup python file
		python /var/www/catalog/catalog/database_setup.py
   
   - sudo service apache2 restart
   
   
  ## prepare the oauth2  google sign in 
  
   - go the https://console.developers.google.com/apis/dashboard  console developer and change the 
   Authorized JavaScript origins and Authorized redirect URIs 
   
   - after u change the urls in google developer
   
   - update file client_secrets.json
   
   - and  update the file __init__.py with new path /var/www/catalog/catalog/client.serets.json 
   
  ## finally
   
   finally visit the web site http://ec2-35-176-122-0.eu-west-2.compute.amazonaws.com
   
   if there is any error  type this command sudo nano /var/log/apache2/error.log and  check errors 
   
   
   
    
   
   
  

   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   


   