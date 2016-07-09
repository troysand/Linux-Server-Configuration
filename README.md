# Linux Server Configuration
## Project Description
This project prepared a Linux server from [Amazon Web Services][AWS] to host a web application. The server was prepared as the final submission for the Full Stack Web Developer nanodegree from [Udacity][Udacity].The project included creating new users, managing firewall settings, and installation and configuration of the required software.

## Server Details
The server details are as follows:
- __Server Name__: ec2-52-35-4-119.us-west-2.compute.amazonaws.com
- __IP Address__: 52.35.4.119
- __SSH Port__: 2200
- __Remote User__: grader
- __Hosted Web App URL__: http://ec2-52-35-4-119.us-west-2.compute.amazonaws.com/catalog

## Summary of Installed Software and Configuration Changes

### List of Software Installed
1. [Apache HTTP Server][Apache]
    - apache2
    - libapache2-mod-wsgi
2. [PostgreSQL Relational Database][Postgresql]
    - postgresql
    - postgresql-contrib
3. [Python][Python] Modules
    - python-flask ([Flask][Flask] microframework)
    - python-sqlalchemy ([SQLAlchemy][SQLAlchemy] object-relational mapper)
    - python-psycopg2 (PostgreSQL adapter)
    - python-httplib2 
    - python-oauth2client
4. [Git Version Control System][Git]
5. [Catalog Web Application][Catalog App]

### Summary of Configuration Changes
1. Created a new user
    - username: grader
    - sudo permission
2. Upgraded and updated installed software packages
3. Modified SSH configuration
    - Changed port from 22 to 2200
    - Disallowed root login
4. Configured the [Uncomplicated Firewall][UFW]
    - Allow SSH (port 2200)
    - Allow HTTP (port 80)
    - Allow NTP (port 123)
    - Deny everything else
5. Configure local time zone
    - set timezone to UTC
6. Configure PostgreSQL for the Catalog App
    - Create a user called catalog with minimal privileges
    - Give the catalog user a password
    - Create the catalog database
7. Configure Apache to serve the Catalog App as a Python mod_wsgi application

## Detailed Description of Actions Performed

### Initial Login
After following the instructions to create the virtual machine, I logged into the server with the following command:

```ssh -i ~/.ssh/udacity_key.rsa root@52.35.4.119```

### Creating a New User
Then I created a user called grader and gave the new user a password with the following command:

```adduser grader```

Next, I gave the grader sudo permission by creating a new file in ```/etc/sudoers.d/``` called ```grader```. This file contained one line that looked like this: ```grader ALL=(ALL)   PASSWD=ALL```. Then I changed the file permissions for this file:
```chmod 0440 grader```

I copied the file ```authorized_keys``` from the ```/root/.ssh``` directory into the ```/home/grader/.ssh``` directory, so the grader user could log in using key based authentication.

### Software Updates
To make sure I had the latest versions of everything installed on the server, I performed the following two commands:

```
apt-get upgrade
apt-get update
```

### SSH Configuration
Next, I changed the SSH configuration by editing ```/etc/ssh/sshd_config``` and changing the entry ```Port 22``` to ```Port 2200```. I also disallowed root login by editing a line to read: ```PermitRootLogin no```. I restarted the ssh server:
```
service ssh restart
```

When I logged back into the server, I had to use the following command:
```
ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@52.35.4.119
```

### Uncomplicated Firewall
Next, I configured the Uncomplicated Firewall to only allow incoming connections for SSH, HTTP, and NTP:
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable
```

### Time Zone
The server time zone was already set to UTC, but I configured it again anyway with the following command:
```
dpkg-reconfigure tzdata
```

With that command I was able to select UTC from a menu.

### New Software
I then performed a bunch of software installation operations to get Apache, PostgreSQL, Git and all of the Python modules installed.
```
apt-get install apache2
apt-get install libapache2-mod-wsgi
apt-get install postgresql
apt-get install postgresql-contrib
apt-get install git-core
apt-get install python-flask
apt-get install python-sqlalchemy
apt-get install python-psycopg2
apt-get install python-httplib2
apt-get install python-oauth2client
```

### Configuring PostgreSQL
I logged into the ```postqres``` user to configure PostgreSQL for the Catalog App.
```
sudo -i -u postgres
```
I used the ```createuser``` to create a database user called catalog:
```
createuser --interactive
```
In this interactive program I answered 'n' to every question about the new user's privileges.
    - Shall the new role be a superuser? n
    - Shall the new role be allowed to create databases? n
    - Shall the new role be allowed to create more new roles? n
Then I gave the new catalog user a password and created the catalog database:
```
psql postgres
postgres=# \password catalog
postgres=# CREATE DATABASE catalog
postgres=# \q
```

### Installing and Editing the Catalog App
Next, I cloned the Catalog App from the github repository at ```/var/www/catalog```:
```
cd /var/www
git clone http://github.com/troysand/catalog.git
```

I had to create a new file called ```catalog.wsgi`` which contained the following lines:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/catalog')

from catalog_app import app as application

application.secret_key = 'super secret key'

if __name__ == "__main__":
    application.debug = True
    application.run()
```

To create and populate the database for my catalog application, I ran the following python programs:
```
cd /var/www/catalog
python catalog_db_setup.py
python create_categories.py
```

### Configuring Apache
To configure Apache to serve my Catalog App as a WSGI application, I had to edit the ```/etc/apache2/sites-available/000-default.conf``` file. I created the following line:
```
WSGIScriptAlias / /var/www/catalog/catalog.wsgi
```

## Resources
The following resources were helpful in answering questions about this project:

### SSH Documentation
- https://help.ubuntu.com/community/SSH/OpenSSH/InstallingConfiguringTesting
- http://www.howtogeek.com/howto/linux/security-tip-disable-root-ssh-login-on-linux/

### UFW Documentation
- https://help.ubuntu.com/community/UFW

### Configuring Local Time Zone
- http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442

### PostgreSQL
- https://help.ubuntu.com/community/PostgreSQL
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04
- http://postgresapp.com/documentation/configuration-python.html
- http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html

### Apache HTTP Server
- https://httpd.apache.org/docs/2.2/

[Udacity]:https://www.udacity.com
[AWS]:http://aws.amazon.com/
[Apache]:http://httpd.apache.org/
[Postgresql]:https://www.postgresql.org/
[Python]:https://www.python.org/
[Flask]:http://flask.pocoo.org/
[SQLAlchemy]:http://www.sqlalchemy.org/
[Git]:https://git-scm.com/
[Catalog App]:https://github.com/troysand/catalog
[UFW]:https://wiki.ubuntu.com/UncomplicatedFirewall
