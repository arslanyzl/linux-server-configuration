# Linux-Server-Configuration

Visit deployed Web App: http://13.58.112.171/

## Project Overview
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

### 1 & 2 - Start a new Ubuntu Linux server instance on Amazon Lightsail

1. Create new instance on Amazon Lightsail. [Udacity](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175462/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e)
2. Download private keys from Amazon Lightsail.
3. Move the private key file on local machine into the folder: `~/.ssh`
4. Set file rights:  
  `$ chmod 600 ~/.ssh/name-of-privet-key`
5. SSH into the instance:  
  `$ ssh -i ~/.ssh/name-of-privet-key.rsa ubuntu@13.58.112.171`
  
### 3. Update installed packages.

1. Update the list of available packages:  
  `$ sudo apt-get update`
2. Install newer vesions of packages you have:  
  `$ sudo sudo apt-get upgrade`

### 4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.

1. Open the config file:  
    `$ sudo nano /etc/ssh/sshd_config`  
2. Change to Port 2200.
3. Change `PermitRootLogin` to `no`.
4. Add `AllowUsers NEWUSER`.  

### 5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
         $ sudo ufw allow 2200/tcp
	 $ sudo ufw allow 80/tcp
	 $ sudo ufw allow 123/udp
	 $ sudo ufw enable   
	 
### 6. Create a new user account named grader.
	$ sudo adduser grader

### 7. Give grader the permission to sudo.
	$ sudo nano /etc/sudoers.d/grader
Add the following text 'grader ALL=(ALL:ALL) ALL'

### 8. Create an SSH key pair for grader using the ssh-keygen tool.
1. On your local machine run: 
	'$ ssh-keygen'
2. Genarated key located in your local machine .shh directory, copy contain of this file to '/home/grader/.ssh/authorized_keys' 
3. Restart ssh:  `sudo service ssh restart`
4. Login using: `ssh -i ~/.ssh/name-of-keygen.rsa -p 2200 grader@13.58.112.171`
	
### 9. Configure the local timezone to UTC.
 	$ sudo dpkg-reconfigure tzdata

### 10. Install and configure Apache to serve a Python mod_wsgi application.
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

### 11. Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql postgresql-contrib`
2. Create linux user for psql:  
  `$ sudo adduser catalog` (choose a password)
3. Change user to postgres:  
  `$ sudo su - postgres`
4. Connect to the system by typing:  
  `$ psql`
- `# CREATE USER catalog WITH PASSWORD 'password';` 
- `# ALTER USER catalog CREATEDB;`
- `# CREATE DATABASE catalog WITH OWNER catalog;`
- `# \c catalog` 
- `# REVOKE ALL ON SCHEMA public FROM public;`
- `# GRANT ALL ON SCHEMA public TO catalog;`
- `# \q`
- `$ exit` 

### 12. Install git, Clone the Catalog app from Github
1. Install git: `sudo apt-get install git`
2. Setup for deploying a Flask Application on Ubuntu VPS
-  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
-  `$ sudo a2enmod wsgi`
-  `$ cd /var/www`
-  `$ sudo mkdir catalog`  
-  `$ cd catalog` 
-  `$ sudo mkdir catalog`  
-  `$ cd catalog` 
-  `$ sudo mkdir static templates`  
-  `$ sudo nano __init__.py`
-  Paste in the following code:  
    ```from flask import Flask  
      app = Flask(__name__)  
      @app.route("/")  
      def hello():  
        return "Hello App"  
      if __name__ == "__main__":  
        app.run() ```  
4. Install Flask
  - `$ sudo apt-get install python-pip` 
  - `$ sudo pip install virtualenv`
  - `$ sudo pip install sqlalchemy`
  - `$ sudo virtualenv venv`
  - `$ source venv/bin/activate`
  - `$ pip install httplib2`
  - `$ pip install requests`
  - `$ pip install Flask`
  - `$ deactivate`
5. New Virtual Host Configuration#
 - `$ sudo nano /etc/apache2/sites-available/catalog.conf`
 - Paste in the following lines of code and change names and addresses regarding your application:  
  ```
    <VirtualHost *:80>
        ServerName 13.58.112.171
        ServerAdmin admin@13.58.112.171
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
  ```
 - `$ sudo a2ensite catalog`
 - `$ cd /var/www/catalog` 
 - `$ sudo nano catalog.wsgi`
 - Paste in the following:  
  ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")
    
    from catalog import app as application
    application.secret_key = 'secret-key'
  ```
  - `$ sudo service apache2 restart`
6. Clone project from GitHub:
 `$ git clone https://github.com/arslanyzl/catalog-item.git`
 Move all to `/var/www/catalog/catalog/`
7. Change for all the line starting with "engine" to (fill in a password the same as for catalog user password):  
  ```engine = create_engine('postgresql://catalog:password@localhost/catalog')```
 - `$ sudo service apache2 restart`
 
 ## Referances
 - [Udacity](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379)
 
 - [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
 
