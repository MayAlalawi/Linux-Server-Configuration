# Linux-Server-Configuration


## Project Overview
Using a baseline installation of a Linux server and preparing it to host my DogsCatalog web application. This includes securing it against a number of attack vectors, installing and configuring a database server and deploying a handmade web application. 

## Why this Project?

This project helo in getting a deep understanding of exactly what the web applications are doing, how they are hosted, and the interactions between multiple systems are what help us be a Full Stack Web Developer. In this project, I will turne a brand-new, bare bones, Linux server into the secure and efficient web application host my applications need.

#### Link to the hoseted web application: <a href = "https://github.com/MayAlalawi/DogsCataloge.git">DogsCatalog</a>
  - Public IP Address: 54.254.245.67
  - Accessible SSH port: 2200
  - SSH Key: I included the private key in the "Notes to Reviewer" section when I submitted my project.
  - URL to hosted web application: http://ec2-54-254-245-67.ap-southeast-1.compute.amazonaws.com/

## Steps to Configure Linux server
#### <strong>Update all currently installed packages</strong>

 ```
 $ sudo apt update     # update available package lists
 $ sudo apt upgrade    # upgrade installed packages
```

#### Change the SSH port on my instance from 22 to 2200
```
  $ sudo nano /etc/ssh/sshd_config
```

#### Configure the UFW

```
  $ sudo ufw status    # inactive
  $ sudo ufw default deny incoming
  $ sudo ufw allow outgoing
  $ sudo ufw allow 2200/tcp
  $ sudo ufw allow www
  $ sudo ufw allow ntp 
  $ sudo ufw enable
  $ sudo ufw status    # active

```

#### Give grader user access
- Create a new user named grader.
  ```$ sudo adduser grader```
- Give sudo access to grader by adding the line ```grader ALL=(ALL:ALL) ALL ``` to the file``` /etc/sudoers.d/grader```.

```$ sudo nano /etc/sudoers.d/grader```
- Create key pair for the grader as specified in Udacity course using ```ssh-keygen ```. and copy it into the server.

#### Disable root login and force authentication using the key pair
- In the file '/etc/ssh/sshd_config' cahnge ```"PermitRootLogin without-password"``` to ```"PermitRootLogin no"``` and uncommented the line that reads ```"PasswordAuthentication no"```.
-  Run ```$ sudo service ssh restart```.

#### Configure the local timezone to UTC
```$ sudo timedatectl set-timezone UTC```

#### Install Apache and mod_wsgi 
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
$ sudo apt-get install python-setuptools
$ sudo service apache2 restart
```
#### Install and configure PostgreSQL
_Install PostgreSQL_
```
$ sudo apt-get install postgresql
$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```
_configure PostgreSQL_
```
$ sudo su - postgres
postgres $ psql
postgres=# CREATE DATABASE dogsdb;
# CREATE USER dogsdb;
# ALTER ROLE dogsdb WITH PASSWORD 'dogsdb'
# GRANT ALL PRIVILEGES ON DATABASE dogsdb TO dogsdb;
```

#### Clone and setup DogsCatalog project from the Github repository
```
$ sudo apt-get install git      #install git to clone repo.

$ cd /var/www
/var/www $ mkdir catalog
/var/www $ cd catalog
/var/www/catalog $ git clone https://github.com/MayAlalawi/DogsCataloge.git
$ cd /var/www/catalog/DogsCataloge
```

Into that directory I cloned my GitHub item catalog project, renamed 'project.py' '__init__.py', changed the url inside the create_engine calls in both the all files to reflect the use of PostgreSQL: "create_engine('postgresql://dogsdb:dogsdb@localhost/dogsdb')"

#### Install Flask and dependencies
```
sudo apt install python3-pip
sudo pip3 install --upgrade pip
sudo pip3 install Flask
sudo pip3 install httplib2
sudo pip3 install requests
sudo pip3 install oauth2client
sudo pip3 install sqlalchemy
```
#### Run the database 
```
 $ python database_setup.py
 $ python listofdogs.py
```


#### Create .wsgi file
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/DogsCataloge/")

from __init__ import app as application
application.secret_key = 'super_secret_key'

```

#### configure Apache to handle requests using the WSGI module
<code>$ sudo nano /etc/apache2/sites-available/catalog.conf</code>

```
<VirtualHost *:80>
                ServerName 54.254.245.67
                ServerAdmin mayalalwi@gmail.com
                ServerAlias HOSTNAME ec2-54-254-245-67.ap-southeast-1.compute.a$
                WSGIScriptAlias / /var/www/catalog/DogsCataloge/catalog.wsgi
                <Directory /var/www/catalog/DogsCataloge/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/DogsCataloge/static
                <Directory /var/www/catalog/DogsCataloge/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

#### Restart Apache to launch the app
<code>$ sudo service apache2 restart </code>


### List of any third-party resources

- <a href="https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps">Digital Ocean</a>
- <a href="https://docs.aws.amazon.com/lightsail/index.html#lang/en_us">Amazon Lightsail Documentation</a>
- <a href="https://hk.saowen.com/a/0a0048ca7141440d0553425e8df46b16cdf4c13f50df4c5888256393d34bb1b9">Deploying python Flask web app on Amazon Lightsail</a>
- <a href="https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04">Initial Server Setup with Ubuntu 16.04</a>

