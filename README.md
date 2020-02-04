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
  sudo ufw allow 80/tcp               # Allow HTTP traffic in.
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
-On AWS instance:  
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

8.Install Apache application and wsgi module
- Run `sudo apt-get install apache2` to install apache.
- Run `sudo apt-get install python-setuptools libapache2-mod-wsgi` to install mod-wsgi module
- Then start the server `sudo service apache2 start`.



  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
