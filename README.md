# Linux-Server-Configuration

This project is the last part of the `Fullstack Web Development Nanodegree`, it ensures that you have solid foundation in configuring and securing linux servers and have the experience to deploy applications using apache web server.


## About
This project is a baseline installation of a Linux server and preparing it to host my web applications. securing my server from a number of attack vectors, installing and configuring a database server, and deploying one of my existing web applications onto it.

# Server details
IP address: `18.195.124.124`

SSH port: `2200`

Item Catalog URL:  **you can access the app via**    
	[http://18.195.124.124.xip.io](http://18.195.124.124.xip.io) **OR** 
[http://ec2-18-195-124-124.eu-central-1.compute.amazonaws.com/](http://ec2-18-195-124-124.eu-central-1.compute.amazonaws.com/)


# Configuration changes
## Add user
Add user `grader` with command: `sudo adduser grader`

## Add user grader to sudo group
Assuming your Linux distro has a `sudo` group (like Ubuntu 16.04), simply add the user to
this admin group:
```
sudo usermod -a -G sudo grader
```

## Update all currently installed packages

- Update the package indexes `apt-get update` 

- To actually upgrade the installed packages `apt-get upgrade` 


## Set-up SSH keys for user grader
As root user do:
```
mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 700 /home/grader/.ssh
cp /ubuntu/.ssh/authorized_keys /home/grader/.ssh/
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```

Can now login as the `grader` user using the command:
`ssh grader@18.195.124.124 -p 2200 -i ~/.ssh/grader_rsa`


## Disable root login
Change the PasswordAuthentication line to no instead of yes in the file `sudo nano /etc/ssh/sshd_config`:

- So it reads:
```
PasswordAuthentication no
```
Do ` sudo service ssh restart` for the changes to take effect.


## Change timezone to UTC
Check the timezone with the `date` command. This will display the current timezone after the time.
If it's not UTC change it like this:

`sudo timedatectl set-timezone UTC`

## Add SSH port 2200

- Reconfigure the port for the ssh server:
`sudo nano /etc/ssh/sshd_config`
- and add:
`port 2200`

- Then reload the configuration:
`sudo service ssh force-reload`

- and connect via:

`ssh grader@18.195.124.124 -p 2200 -i ~/.ssh/grader_rsa`


## Configuration Uncomplicated Firewall (UFW)
By default, block all incoming connections on all ports:

`sudo ufw default deny incoming`

- Allow outgoing connection on all ports:

`sudo ufw default allow outgoing`

- Allow incoming connection for SSH on port 2200:

`sudo ufw allow 2200/tcp`

- Allow incoming connections for HTTP on port 80:

`sudo ufw allow www`

- Allow incoming connection for NTP on port 123:

`sudo ufw allow ntp`

- To check the rules that have been added before enabling the firewall use:

`sudo ufw show added`

- To enable the firewall, use:

`sudo ufw enable`

- To check the status of the firewall, use:

`sudo ufw status`

## Install Apache to serve a Python mod_wsgi application
- Install Apache:

`sudo apt-get install apache2`

- Install the `libapache2-mod-wsgi` package:

`sudo apt-get install libapache2-mod-wsgi`

## Install and configure the PostgreSQL database system
 ```
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```
 
## Create a PostgreSQL role named catalog and a database named catalog
```
sudo -u postgres -i
postgres:~$ createuser catalog
postgres:~$ createdb catalog
postgres:~$ psql catalog
postgres=# ALTER DATABASE catalog OWNER TO catalog;
postgres=# ALTER USER catalog WITH PASSWORD 'catalog';
postgres=# \q
postgres:~$ exit

```

## Install Flask, SQLAlchemy, etc
- Issue the following commands:
```
sudo apt-get install python3-psycopg2 python3-flask
sudo apt-get install python3-sqlalchemy python3-pip
sudo pip3 install --upgrade pip
sudo pip3 install oauth2client
sudo pip3 install requests
sudo pip3 install httplib2
sudo pip3 install flask-seasurf
```


## Install Git version control software
`sudo apt-get install git`

## Clone the repository that contains Item Catalog app

```
cd /var/www
sudo mkdir item-project
sudo chown -R grader:grader item-project
cd project4
sudo git clone https://github.com/hima-Megahed/Item-Catalog itemcatalogproject
```

## Configure the Catalog app
-  Add a new empty module named `__init__.py`
-  Provide the full path (or location) to all the files, instead of just the filename

## Configure Google OAuth
- Add the public IP address of your server to the “Authorized javascript origins”
- Add the public IP address to the client secrets JSON file

## Configure the web application to connect to the PostgreSQL database instead of a SQLite database
 ```
 sudo nano /var/www/item-catalog/itemcatalogproject/controller.py
 -Change the engine setting in the file from 'sqlite:///itemCatalogs.db' to 'postgresql://catalog:catalog@localhost/catalog', and save
``` 

## Configure the Database to connect to the PostgreSQL database instead of a SQLite database
  ```
 sudo nano /var/www/item-catalog/itemcatalogproject/models.py
 -Change the engine setting in the file from 'qlite:///sitemCatalogs.db' to 'postgresql://catalog:catalog@localhost/catalog', and save
 ```
 
 ## Configure Apache to serve the web application using WSGI
- Create the web application WSGI file.
 ```
sudo nano /var/www/item-catalog/app.wsgi
```
- Add the following lines to the file, and save the file.
```
#!/usr/bin/python3
import sys 
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/item-catalog/itemcatalogproject/")
from controller import app as application
application.secret_key = 'super_secret_key'

```
- Update the Apache configuration file to serve the web application with WSGI.
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
- Add the following line inside the `<VirtualHost *:80>` element, and save the file.
```
        ServerName 18.195.124.124
	ServerAdmin Ibrahim__Hasan@outlook.com
	WSGIScriptAlias / /var/www/item-catalog/app.wsgi
	<Directory /var/www/item-catalog/itemcatalogproject/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/item-catalog/itemcatalogproject/static
	<Directory /var/www/item-catalog/itemcatalogproject/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
 
```

- Restart Apache.
```
sudo apache2ctl restart
```

## References
- [Ask Ubuntu](http://askubuntu.com/)
- Stackoverflow and Udacity Forums.

## Attribution 
- **[Basma Ashour](https://github.com/basmaashouur)** Thanks for a neat and straightforward README
