# Introduction 

this is a document for Linux server configuration hosted on Amazon Lightsail
IP address: http://18.197.159.3/
SSH port: 2200 

## Steps: 
- create an instance on Amazon Lightsail
- update all the packages
- changed the ssh port from 22 to 2200 
- enabled the firewall to be able to only recieve requests using those ports
    - SSH (port 2200)
    - HTTP (port 80)
    - NTP (port 123)

- create user name called: grader 
- give him the sudo privilage 
- create ssh key pair to be able to login 
- log in as grader using:
```sh
$ ssh -p 2200 grader@18.197.159.3 -i ~/.ssh/graderkey
```
- configure the timezone 
```sh
$ sudo dpkg-reconfigure tzdata
```
- Install and configure appache 
```sh
$ sudo apt-get install apache2
```
- Install mod_wsgi to run a flask application 
```sh
$ sudo apt-get install libapache2-mod-wsgi python-dev
```
enable that module 
```sh
$ sudo a2enmod wsgi
```
- Install PostgreSQL
    - create a new PostgreSQL user called: catalog 
    - change the user to postgres
```sh
$ sudo su - postgres
```

- connect to psql and create the user 

```sh
$ CREATE ROLE catalog WITH LOGIN;
```
- give catalog user the privilage to create databases 
```sh
$ ALTER ROLE catalog CREATEDB;
```
create a linux user called catalog and give him the sudo privilage
login as catalog to create a database called catalog 
```sh
$ createdb catalog
```
- install git 
- clone the catalog project 
 ```sh
$ sudo git clone https://github.com/MaiMahmoudyoussef/catalog.git 
```
- give ubuntu ownership to this directory 
 ```sh
$ sudo chown -R ubuntu:ubuntu catalogproject/ 
```
### Modify catalog project configurations 
- change the name of the application python file to  __ init__.py
- change the app.run function in  __ init__.py  
 ```sh
$ app.run()
```
- switch the database from SQLite to PostgreSQL by replacing the engine function by the below function; in the three files (__ init__.py , database_setup.py ,  lotsofcatalog .py )
 ```sh
$ engine = create_engine('postgresql://catalog:PASSWORD_FOR_DATABASE_HERE@localhost/catalog')
```
### Setup a vitual environment and install packages
- make sure that you have installed pip
 ```sh
$ sudo apt-get install python-pip
```
- Install virtualenv
 ```sh
$ sudo apt-get install python-virtualenv
```
- change the directory to  /var/www/catalogproject/projectcatalog
- active the new environment
 ```sh
$ source ./venv/bin/activate
```
- install all the python packages
 ```sh
$ pip install httplib2

$ pip install requests

$ pip install --upgrade oauth2client

$ pip install sqlalchemy

$ pip install flask

$ pip install psycopg2
```
- if you faced that error `locale.Error: unsupported locale setting`  
Run following commands
 ```sh
$ export LC_ALL="en_US.UTF-8"
$ export LC_CTYPE="en_US.UTF-8"
$ sudo dpkg-reconfigure locales
```
- deactivate the virtal environment  
```sh
$ deactivate
```
### Set up and enable a virtual host
- Create a file in /etc/apache2/sites-available/ called catalogproject.conf
- add this code into the file 
```sh
<VirtualHost *:80>
		ServerName xx.xx.xx.xx
		ServerAdmin mayXXX@gmail.com
		WSGIScriptAlias / /var/www/catalogproject/catalogproject.wsgi
		<Directory /var/www/catalogproject/catalogproject/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalogproject/catalogproject/static
		<Directory /var/www/catalogproject/catalogproject/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- enable the virtual host
```sh
$ sudo a2ensite catalogproject
```
### Create a wsgi file 
- create a file called catalogproject.wsgi in /var/www/catalogproject directory 
- add the following to the file 
```sh
activate_this = '/var/www/catalogproject/catalogproject/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalogproject/")

from catalogproject import app as application
application.secret_key = '12345'
```
### Set up the database schema
- in the /var/www/catalogproject/catalogproject/ directory active the virtualenv
- run lotsofcatalog. py to add entries to your schema 
- then deactivate 

