# Linux Server Configuration


This project is for installation of a Linux server and prepare it to host my web applications. And will secure the server from a number of attack vectors, install and configure a database server, and deploy my web applications [Item Catalog](https://github.com/islamsalah2020/Item_Catalog) onto it using Lightsail AWS instance as virtual machine.

## Configuration Steps:

1. Create a new Ubuntu Linux server instance on AWS.
- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an AWS account.
- Once you are login into the console, click `Create instance`.
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 18.04 LTS`.
- Keep the default name provided by AWS then click the `Create` button to create the instance.
- Save private key file named `My_ssh_key.pem` into the local machine in folder `~/.ssh` then change name to lightsail_key.rsa.

2. SSH into the server 
- Open terminal, type: `chmod 600 ~/.ssh/My_ssh_key.pem`.
- To connect to the instance via the terminal: type `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.194.150.68`, 
  where `18.194.150.68` is the public IP address of the instance.

3. Update and upgrade installed packages.
```
sudo apt-get update
sudo apt-get upgrade
```

4. Configure SSH port and the Uncomplicated Firewall (UFW)
- Edit `/etc/ssh/sshd_config` file by `sudo nano /etc/ssh/sshd_config` Change port from 22 to 2200.
- for UFW :
```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow 80/tcp            # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
  ```
- Restart SSH service with `sudo service ssh restart`.
- Turn UFW on `sudo ufw enable` then check status `sudo ufw status` it should be active now.
- Exit the SSH connection: `exit`.
- Reconfigure Netowrk in lightsail instance:
  ```
   SSH	TCP	22	
   HTTP	TCP	80	
   Custom	UDP	123	
   Custom	TCP	2200
   ```
   
5. Create a new user called grader and give an access 
- Connect to the instance via the terminal: type `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@18.194.150.68`.
- To create a new user called `grader` Run `sudo adduser grader`.
- Create a new directory in sudoer directory with name `sudo nano /etc/sudoers.d/grader`.
- Add `grader ALL=(ALL:ALL) ALL` using nano editor.

6. Create an SSH key pair for `grader` using the `ssh-keygen` tool
- On the local machine:
  - Run `ssh-keygen`.
  - Enter file in which to save the key (I gave the name `grader_key`) in the local directory `~/.ssh`.
  - Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`).
  - Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file.
- On AWS instance:  
  Run the following command in your virtual environment:
  ```
  su - grader
  mkdir .ssh
  touch .ssh/authorized_keys
  nano .ssh/authorized_keys 
  ```
  then paste your generated public SSH key here.
- Reload SSH with `service ssh restart`.
- Then exit from the AWS instance and login as grader user:
```ssh -i ~/.ssh/grader_key -p 2200 grader@18.194.150.68```

## Prepare to deploy the project
7. Configure the local timezone to UTC
- Run `sudo dpkg-reconfigure tzdata` and choose UTC.

8. Install Apache application and wsgi module
- Run `sudo apt-get install python`
- Run `sudo apt-get install apache2` to install apache.
- Run `sudo apt-get install python-setuptools libapache2-mod-wsgi` to install mod-wsgi module
- Then start the server `sudo service apache2 start`.

9. Install git
- Run `sudo apt-get install git`.
- Configure your username and email. `git config --global user.name <username>` and `git config --global user.email <email>`

10. Clone the project
- While logged in as grader:
```
mkdir /var/www/catalog/
cd /var/www/catalog
touch catalog.wsgi
sudo git clone https://github.com/islamsalah2020/Item_Catalog
```
- Rename application.py to __init__.py  
```mv application.py  __init__.py```

11. Create virtual environment and install Flask framework
- Install pip 
```sudo apt-get install python-pip```
- Install virtual environment, Run `sudo apt-get install python-virtualenv`. 
- Create a new virtuall environment with name ### venv, Run `sudo virtualenv venv`.
- Change permissions to the viertual environment folder sudo chmod -R 777 venv.
- Activate virtuall environment, Run `source venv/bin/activate`.
- Install Flask Run `pip install Flask`.
- Install Flask dependencies `pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2`.

12. Configure Apache
- Create a config file, Run `sudo nano /etc/apache2/sites-available/catalog.conf`.
- Add the following lines to configure the virtual host:
```
WSGIPythonHome "/usr/local/bin"
WSGIPythonPath "/home/fenikso/virtualenv/lib/python3.4/site-packages"

<VirtualHost *:80>
    ServerName 13.59.39.163
  ServerAlias ec2-13-59-39-163.us-west-2.compute.amazonaws.com
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
- Enable virtual host, Run `sudo a2ensite catalog`.
- restart Apache, Run `sudo service apache2 restart`.

13. Set up the Flask application with WSGI
- Create catalog.wsgi file, Run `touch /var/www/catalog/catalog.wsgi` then add the following lines:
```
activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "..."
```
- Restart Apache, Run `sudo service apache2 restart`.

14. Install and configure PostgreSQL
- Install Postgresql, Run `sudo apt-get install PostgreSQL`.
- Login to postgresql, Run ` sudo su - postgres and psql`.
- Create a new user, Run `CREATE USER catalog WITH PASSWORD 'password'`.
- Create a DB named 'catalog', Run `ALTER USER catalog CREATEDB` and `CREATE DATABASE catalog WITH OWNER catalog`.
- Connect to the DB, Run `\c catalog`.
- Revoke all rights, Run `REVOKE ALL ON SCHEMA public FROM public`.
- Change a grant from public to catalog, Run `GRANT ALL ON SCHEMA public TO catalog`.
- Logout from postgresql prompt and return to the grader user, Run `\q` and `exit`.
- Replace the engine inside Flask application in database_setup.py in the following line:
```engine = create_engine('postgresql://catalog:password@localhost/catalog')```
with :
```Set up the DB with python /var/www/catalog/item-catalog-udacity/database_setup.py```
- Run `sudo service apache2 restart` and check the website.

  
  
  
  
  
  
  
  
  
  
  
  
  
  
