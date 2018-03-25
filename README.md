# udacity-linux-server-configuration

### Installation
Summary of software installed and configuration changes made.

Update all currently installed packages
```sh
$ sudo apt update     # update available package lists
$ sudo apt upgrade    # upgrade installed packages
$ sudo apt autoremove # automatically remove packages that are no longer require
```

# Add new user grader and add to sudo group
https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
```sh
$ sudo adduser grader     # add user grader

Set password prompts:
Enter new UNIX password: # grader
Retype new UNIX password: # grader
passwd: password updated successfully

$ sudo usermod -aG sudo grader #Use the usermod command to add the user to the sudo group.
```

# Change the SSH port from 22 to 2200.
```sh
sudo nano /etc/ssh/sshd_config  # change port 22 to 2200
sudo service ssh restart        # restart ssh service
```
# Create an SSH key pair for grader using the ssh-keygen tool
>https://www.youtube.com/watch?v=vHKOVt4cefA
# local machine tools
* putty
* puTTYgen
* WinSCP
# 
>  use an empty passphrase when prompt
```sh
sudo mkdir /home/grader/.ssh
sudo chown grader:grader /home/grader/.ssh # changing ownership of .ssh to grader
sudo chmod 700 /home/grader/.ssh           # change folder permission
sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/
sudo chmod 644 /home/grader/.ssh/authorized_keys
sudo service ssh restart        # restart ssh service
```
# Use Bash or Putty to login with Grader and PEM or PPK key
##### BASH
>.pem file
ssh -i ~/Downloads/grader.pem grader@18.222.4.106 -p 2200
##### Putty
>.ppk
# Disable root
```
sudo nano /etc/ssh/sshd_config  # open sshd_config
change PermitRootLogin to no
sudo service ssh restart        # restart ssh service
```
## Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
> Warning: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server.
When you change the SSH port, the Lightsail instance will no longer be accessible through the web  'Connect using SSH' button. The button assumes the default port is being used.
```sh
sudo ufw status                 # check ufw status 
sudo ufw default deny incoming  # initially block all incoming requests
sudo ufw default allow outgoing # default rule for outgoing connections
sudo ufw allow 2200/tcp         # allow SSH on port 2200
sudo ufw allow www              # allow HTTP on port 80
sudo ufw allow ntp              # allow NTP on port 123
sudo ufw enable                 # enable firewall
sudo ufw status                 # check ufw status

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123                        ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123 (v6)                   ALLOW       Anywhere (v6)

```

# Configure the local timezone to UTC.
> https://www.digitalocean.com/community/tutorials/how-to-set-up-time-synchronization-on-ubuntu-16-04
```
sudo timedatectl set-timezone UTC
```


# Install Apache and mod_wsgi for python3
```sh
sudo apt install apache2                  # install apache
sudo apt install libapache2-mod-wsgi-py3  # install python3 mod_wsgi
```
# Install and configure PostgreSQL
Install PostgreSQL with:
```sh
sudo apt-get install postgresql postgresql-contrib

To ensure that remote connections to PostgreSQL are not allowed, check that the configuration file /etc/postgresql/9.5cd/main/pg_hba.conf only allowed connections from the local host addresses 127.0.0.1 for IPv4 and ::1 for IPv6.

Create a PostgreSQL user called catalog with:

sudo -u postgres createuser -P catalog

You are prompted for a password. This creates a normal user that cant create databases, roles (catalog).

Create an empty database called catalog with:

sudo -u postgres createdb -O  catalog catalog   #{uppercse letter O}

The Ubuntu documentation page on PostgreSQL was helpful.
```

```sh
Alternate way to Install and configure PostgreSQL
Install PostgreSQL sudo apt-get install postgresql

Check if no remote connections are allowed sudo vi /etc/postgresql/9.3/main/pg_hba.conf

Login as user "postgres" sudo su - postgres

Get into postgreSQL shell psql

Create a new database named catalog and create a new user named catalog in postgreSQL shell

postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
Set a password for user catalog

postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
Give user "catalog" permission to "catalog" application database

postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Quit postgreSQL postgres=# \q

Exit from user "postgres"

exit

```

>To reset the password if you have forgotten:
>ALTER USER user_name WITH PASSWORD 'new_password';
sudo python3 database_setup




# Install packages: Flask and SQLAlchemy
Issue the following commands:
~~sudo apt-get install python-psycopg2 python-flask~~~
~~sudo apt-get install python-sqlalchemy python-pip~~
~~sudo apt-get -qqy install postgresql python-psycopg2~~
~~del sudo pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests~~
~~sudo pip install flask packaging oauth2client redis passlib flask-httpauth~~
~~sudo pip install --upgrade pip~~
~~sudo pip install oauth2client~~
~~sudo pip install requests~~
~~sudo pip install httplib2 (not necessary, already installed)~~
```sh
Python 3
sudo apt-get install python3-psycopg2
sudo apt-get pip3 install flask
sudo apt-get install python3-sqlalchemy
sudo apt-get install python3-oauth2client
```
# Install Git
```
sudo apt-get install git

```


# Clone and setup your Item Catalog project from the Github repository
```sh
sudo mkdir /var/www/catalog # create catalog folder
###sudo chown -R grader:grader catalog
cd /var/www/catalog
sudo git clone https://github.com/eh4269/CaseManager.git
```

# create .wsgi file
```sh
sudo nano project.wsgi
# from project import app as application
```
### Add the following lines of code to the flaskapp.wsgi file:
```sh
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```
# Rename Catalog app.py file so we can import it as a package
```sh
(remote) grader$ cd /var/www/catalog/CaseManager
(remote) grader$ mv app.py __init__.py
```
## Add WSGI configuration for the Catalog project
```sh
(remote) grader$ sudo nano /etc/apache2/sites-enabled/000-default.conf

<VirtualHost *:80>
    ServerName 18.222.4.106
    ServerAdmin eh4269@hotmail.com
        WSGIScriptAlias / /var/www/catalog/project.wsgi
        <Directory /var/www/catalog/CaseManager>
                WSGIProcessGroup CaseManager
                WSGIApplicationGroup %{GLOBAL}
                Order deny,allow
                Allow from all
        </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```






##### Python 3 relative path issue
> https://stackoverflow.com/questions/44801661/init-py-does-not-find-modules-in-same-directory
pyhton 2 vs python 3 relative path issue
Your code would work for Python 2.x, but not 3.x because of different relative imports syntax: without dot, Python 2.x would look for modules in module root and current package, and Python 3.x will look only in module root.

>Import statements you want to use are these:

>from binaryio.binaryreader import BinaryReader
>from binaryio.binarywriter import BinaryWriter
>Works in both Python 2.x and 3.x without "futures"



# Change the relative link for the client_secret.json to absolute
```sh
In application.py, change:

client_id = json.loads(open('client_secret.json', 'r').read())['web']['client_id']
To:

client_id = json.loads(open('/var/www/catalog/catalog/client_secret.json', 'r').read())['web']['client_id']
```

# Change the Catalog database to use PostgreSQL instead of SQLite
```sh
In database_setup.py, application.py, and feedme.py, change:

engine = create_engine('sqlite:///catalog.db')
To:

engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```

# Reload the Apache configuration
```
(remote) grader$ sudo service apache2 reload
```

# View in your browser!
># IP 18.222.4.106
># www.edwardhernandezonline.com


