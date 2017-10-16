# Linux-Server-Configuration

Visit deployed Web App: http://13.58.112.171/

## Project Overview
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

### 1 & 2 - Start a new Ubuntu Linux server instance on Amazon Lightsail

1. Create new instance on Amazon Lightsail. [Udacity](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175462/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e)
2. Download private keys from Amazon Lightsail.
3. Move the private key file on local machine into the folder: ~/.ssh
4. Set file rights:  
  `$ chmod 600 ~/.ssh/name-of-privet-key`
5. SSH into the instance:  
  `$ ssh -i ~/.ssh/name-of-privet-key.rsa ubuntu@PUPLIC-IP-ADDRESS`
  
### 3. Update all currently installed packages.

1. Update the list of available packages:  
  `$ sudo apt-get update`
2. Install newer vesions of packages you have:  
  `$ sudo sudo apt-get upgrade`

### 4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.

1. Open the config file:  
    `$ sudo nano /etc/ssh/sshd_config`  
2. Change to Port 2200.
3. Change `PermitRootLogin` from `without-password` to `no`.
4. Append `AllowUsers NEWUSER`.  

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
3. Restart ssh with `sudo service ssh restart`
4. Now you are only able to login using `ssh -i ~/.ssh/name-of-keygen.rsa -p 2200 grader@PUPLIC-IP-ADDRESS`
	
### 9. Configure the local timezone to UTC.
Configure the time zone `sudo dpkg-reconfigure tzdata`

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
  - `$ sudo virtualenv venv`
  - `$ sudo chmod -R 777 venv`
  - `$ source venv/bin/activate`
  - `$ pip install httplib2`
  - `$ pip install requests`
  - `$ pip install Flask`
  - `$ deactivate`
5. Configure and Enable a New Virtual Host#
 - `$ sudo nano /etc/apache2/sites-available/catalog.conf`
 - Paste in the following lines of code and change names and addresses regarding your application:  
  ```
    <VirtualHost *:80>
        ServerName PUBLIC-IP-ADDRESS
        ServerAdmin admin@PUBLIC-IP-ADDRESS
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



#### 11.4 - Install needed modules & packages
1. Activate virtual environment:  
  `$ source venv/bin/activate`
2. Install httplib2 module in venv:  
  `$ pip install httplib2`
3. Install requests module in venv:  
  `$ pip install requests`
4. *Install flask.ext.seasurf (only seems to work when installed globally):  
  `$ *sudo pip install flask-seasurf`
5. Install oauth2client.client:  
  `$ sudo pip install --upgrade oauth2client`
6. Install SQLAlchemy:  
  `$ sudo pip install sqlalchemy`
7. Install the Python PostgreSQL adapter psycopg:  
  `$ sudo apt-get install python-psycopg2`

### 10 - Install and configure PostgreSQL
Source: [DigitalOcean][23] (alternatively, nice short guide on [Kill The Yak][24] as well)  

1. Install PostgreSQL:  
  `$ sudo apt-get install postgresql postgresql-contrib`
2. Check that no remote connections are allowed (default):  
  `$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Open the database setup file:  
  `$ sudo vim database_setup.py`
4. Change the line starting with "engine" to (fill in a password):  
  ```python engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')```  
5. Change the same line in application.py respectively
6. Rename application.py:  
  `$ mv application.py __init__.py`
7. Create needed linux user for psql:  
  `$ sudo adduser catalog` (choose a password)
8. Change to default user postgres:  
  `$ sudo su - postgre`
9. Connect to the system:  
  `$ psql`
10. Add postgre user with password:  
  Sources: [Trackets Blog][25] and [Super User][26]
  1. Create user with LOGIN role and set a password:  
    `# CREATE USER catalog WITH PASSWORD 'PW-FOR-DB';` (# stands for the command prompt in psql)
  2. Allow the user to create database tables:  
    `# ALTER USER catalog CREATEDB;`
  3. *List current roles and their attributes:
    `# \du`
11. Create database:  
  `# CREATE DATABASE catalog WITH OWNER catalog;`
12. Connect to the database catalog
  `# \c catalog` 
13. Revoke all rights:  
  `# REVOKE ALL ON SCHEMA public FROM public;`
14. Grant only access to the catalog role:  
  `# GRANT ALL ON SCHEMA public TO catalog;`
15. Exit out of PostgreSQl and the postgres user:  
  `# \q`, then `$ exit` 
16. Create postgreSQL database schema:  
  $ python database_setup.py

#### 11.5 - Run application 
1. Restart Apache:  
  `$ sudo service apache2 restart`
2. Open a browser and put in your public ip-address as url, e.g. 52.25.0.41 - if everything works, the application should come up
3. *If getting an internal server error, check the Apache error files:  
  Source: [A2 Hosting][27]  
  1. View the last 20 lines in the error log: 
    `$ sudo tail -20 /var/log/apache2/error.log`
  2. *If a file like 'g_client_secrets.json' couldn't been found:  
    Source: [Stackoverflow][28]  

#### 11.6 - Get OAuth-Logins Working
  Source: [Udacity][29] and [Apache][30]  
  
1. Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 52.25.0.41, its ec2-52-25-0-41.us-west-2.compute.amazonaws.com
2. Open the Apache configuration files for the web app:
  `$ sudo vim /etc/apache2/sites-available/catalog.conf`
3. Paste in the following line below ServerAdmin:  
  `ServerAlias HOSTNAME`, e.g. ec2-52-25-0-41.us-west-2.compute.amazonaws.com
4. Enable the virtual host:  
  `$ sudo a2ensite catalog`
5. To get the Google+ authorization working:  
  1. Go to the project on the Developer Console: https://console.developers.google.com/project
  2. Navigate to APIs & auth > Credentials > Edit Settings
  3. add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-52-25-0-41.us-west-2.compute.amazonaws.com/oauth2callback
6. To get the Facebook authorization working:
  1. Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
  2. Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
  3. To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'

#### 11.7** - Install Monitor application Glances
Sources: [Glances][31] and [Web Host Bug][32]

1. `$ sudo apt-get install python-pip build-essential python-dev`
2. `$ sudo pip install Glances`
3. `$ sudo pip install PySensors`

[1]: https://de.wikipedia.org/wiki/Flask "Wikipedia entry to Flask"
[2]: https://github.com/stueken/FSND-P3_Music-Catalog-Web-App "GitHub repository of an item catalog web app"
[3]: https://www.udacity.com/account#!/development_environment "Instructions for SSH access to the instance"
[4]: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps "How To Add and Delete Users on an Ubuntu 14.04 VPS"
[5]: http://askubuntu.com/questions/410244/a-command-to-list-all-users-and-how-to-add-delete-modify-users "How to list, add, delete and modify users"
[6]: http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade "What is the difference between apt-get update and upgrade?"
[7]: https://help.ubuntu.com/community/AutomaticSecurityUpdates "AutomaticSecurityUpdates"
[8]: http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server "Create a new SSH user on Ubuntu Server"
[9]: http://unixhelp.ed.ac.uk/CGI/man-cgi?sshd_config "UNIX man page: SSHD_CONFIG"
[10]: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server "How To Configure SSH Key-Based Authentication on a Linux Server"
[11]: http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none "Error message when I run sudo: unable to resolve host (none)"
[12]: http://superuser.com/questions/815433/how-urgent-is-a-system-restart-required-for-security "How urgent is a *** System restart required *** for security?"
[13]: http://askubuntu.com/questions/483670/what-causes-ssh-problems-after-rebooting-a-14-04-server "What causes SSH problems after rebooting a 14.04 server?"
[14]: https://help.ubuntu.com/community/UFW "UFW - Uncomplicated Firewall"
[15]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04 "How To Install and Use Fail2ban on Ubuntu 14.04"
[16]: https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29 "Ubuntu Time Management"
[17]: http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html "A Step by Step Guide to Install LAMP (Linux, Apache, MySQL, Python) on Ubuntu"
[18]: http://askubuntu.com/questions/256013/could-not-reliably-determine-the-servers-fully-qualified-domain-name "Could not reliably determine the server's fully qualified domain name?"
[19]: https://help.github.com/articles/set-up-git/#platform-linux "Set Up Git for Linux"
[20]: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps "How To Deploy a Flask Application on an Ubuntu VPS"
[21]: http://stackoverflow.com/questions/14695278/python-packages-not-installing-in-virtualenv-using-pip "python packages not installing in virtualenv using pip"
[22]: http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible "Make .git directory web inaccessible"
[23]: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps "How To Secure PostgreSQL on an Ubuntu VPS"
[24]: http://killtheyak.com/use-postgresql-with-django-flask/ "All I want to do is use PostgreSQL with Flask or Django."
[25]: http://blog.trackets.com/2013/08/19/postgresql-basics-by-example.html "PostgreSQL Basics by Example"
[26]: http://superuser.com/questions/769749/creating-user-with-password-or-changing-password-doesnt-work-in-postgresql "Creating user with password or changing password doesn't work in PostgresQL"
[27]: https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files "How to view Apache log files"
[28]: http://stackoverflow.com/questions/12201928/python-open-method-ioerror-errno-2-no-such-file-or-directory "Python: No such file or directory"
[29]: http://discussions.udacity.com/t/oauth-provider-callback-uris/20460 "OAuth Provider callback uris"
[30]: http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html "Name-based Virtual Host Support"
[31]: http://glances.readthedocs.org/en/latest/glances-doc.html#introduction "Glances Documentation"
[32]: http://www.webhostbug.com/install-use-glances-ubuntudebian/ "How to install and use Glances on Ubuntu/Debian"
