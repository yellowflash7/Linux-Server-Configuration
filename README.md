# Linux-Server-Configuration
A guide to configure your own server using amazon lightsail to make catalog-project live

### IP Information
**Ip Adress:-13.126.58.158**   
**SSH port:-2200**   

## How To Setup   
* create an account on amazon lightsail and prepare it host catalog project
* Run the follwing command to update all packages on system
  '''sudo apt-get update && apt-get upgrade'''
* update timezone settings

**Instructions to ssh into your server**
* Download your private key
* Move the private key file into the folder ~/.ssh
* type the follwing commands
   ```chmod 600 ~/.ssh/your_key.rsa```
   ```ssh -i ~/.ssh/udacity_key.rsa root@13.126.58.158```

**Instructions to create a new user**
* ```sudo useradd grader```
* Install finger to verify grader(Optional)
* ```usermod -aG grader sudo```  //give grader sudo permissions

### Securing your server   
**Change the SSH port to 2200**   
* ```sudo nano /etc/ssh/sshd_config``` //edit sshd_config to change port   
* change PermitRootLogin from prohibited-password to no   
* ```sudo service ssh restart```   

**Configure Your Firewall**   

* Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (
port 123)*      
   ```sudo ufw status```     
   ```sudo ufw default deny incoming```   
   ```sudo ufw default allow outgoing```     
   ```sudo ufw allow 2200/tcp```     
   ```sudo ufw allow 80/tcp```     
   ```sudo ufw allow 123/udp```     
   ```sudo ufw enable```        
 
* connect to the  serving using SSH Button because changing the SSH port will revoke your access via web app  
* Download the key pair from amazon lightsail   
* ```chmod 600 key-pair.pem``` // Grant permissions    
* Sort out the initial login issue     
* ```sudo nano /etc/ssh/sshd_config```     
  ```set Password Authentication to "yes" (which we'll change later)```        
  ```sudo service ssh restart```     
* Using ssh-keygen tool generate ssh key      
  ```ssh-keygen```    //Generate ssh key pair        
* Save your keygen file in your ssh directory       
   ```/home/Users//.ssh/item-catalog```           
* You can add a password/passphrase to use in case your keygen file gets compromised          
* Login to your account(grader)      
    ```ssh grader@13.126.58.158 -p 2200 -i key-pair.pem```        
    ```mkdir .ssh```  /// make ssh directory        
    ```touch .ssh/authorized_keys``` //make file to store key       
* Read contents of public key ```cat .ssh/keygen.pub```      
* Paste this key into grader and save  ```nano .ssh/authorized_keys```        
* Set permissions of the .ssh directory and key file      
    ```sh - sudo chmod 700 .ssh - sudo chmod 644 .ssh/authorized_keys```      
* Set password authentication to 'no'    
* Login with key-pair      
   ```ssh grader@Public-IP-Address* -p 2200 -i ~/.ssh/keygen```     
   
### Deploying the Project  
 * Install git  ```sudo apt-get install git```   
 * Install and configure Apache to serve a Python mod_wsgi application.    
    ```sudo apt-get install apache2```    
    ```sudo apt-get install libapache2-mod-wsgi```   
    ```sudo nano /etc/apache2/sites-enabled/000-default.conf``` //Using wsgi module configure apache to handle requests      
    ```WSGIScriptAlias / /var/www/html/myapp.wsgi``` // Add this before closing line     
 * Restart Apache    
    ```sudo apache2ctl restart```    
 *  Verify WSGI enabled and Install python   
    ```sudo apt-get install python-dev```    
    ``` sudo a2enmod wsgi```   
 * Install postgreSQL and configure     
    1. Do not allow remote connections  
    2. Create a new database user named catalog that has limited permissions to your catalog application database.  
 

* Clone and setup your Item Catalog project from the Github repository   
* Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser.  
* Make sure that your .git directory is not publicly accessible via a browser  
* Now, Clone Catalog project   
    ```cd /var/www```    
    ```sudo mkdir catalog```    
    ```cd catalog```    
    ```git clone https://github.com/<your_project_path>/```   
    ```sudo touch catalog.wsgi```   
* Paste the following and save   
    ```#!/usr/bin/python  
 	     import sys   
 	     import logging   
 	     logging.basicConfig(stream=sys.stderr)    
 	     sys.path.insert(0,"/var/www/catalog/")   

 	     from catalog import app as application  
 	     application.secret_key = 'Add your secret key'```  

 * Installing dependencies for catalog project   
    ```sudo apt-get -qqy install postgresql python-psycopg2      
       sudo apt-get install python-pip    
       sudo pip install requests    
       sudo pip install oauth2client    
       sudo pip install htpplib2    
       sudo apt-get -qqy install python-sqlalchemy   
       pip install werkzeug==0.8.3  
       pip install flask==0.9    
       pip install Flask-Login==0.1.3```   

* Enable and Configure new virtual host    
* Create host config file and edit and paste the following     
   ```<VirtualHost *:80>    
        ServerName 13.126.58.158   
        ServerAdmin admin@13.126.58.158     
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
      </VirtualHost>```   
      
* Enable ```sudo a2ensite catalog```   
* Restart apache ```sudo service apache2 restart```   
* Install and setup PostgreSQL  
   ```sudo apt-get install postgresql   
      sudo -i -u postgres   
      psql```     
   ```CREATE USER catalog WITH PASSWORD 'catalog';```   
   ```ALTER USER catalog CREATEDB;```  
   ```CREATE DATABASE catalog WITH OWNER catalog;```   
   ```\c catalog```   
   ```REVOKE ALL ON SCHEMA public FROM public;```   
   ```GRANT ALL ON SCHEMA public TO catalog;```  
   ```exit```  
* Edit and config database_setup.py
   ```sudo nano database_setup.py```    
   ```engine = create_engine('postgresql://catalog:catalog@localhost/catalog')```  
* Edit and config init.py
   ```sudo nano init.py``` 
   ```engine = create_engine('postgresql://catalog:catalog@localhost/catalog')```  


**ADD The public Ip in google developers console**
   

