# linuxconfiguration
Udacity Linux Configuration project 

This project is intended for The graduation project of Full stack nanodegree 

IP Addess Used : 18.194.31.70
SSH port 2200

Project URL : http://18.194.31.70/

conncect via SSH by 
ssh -i ~/.ssh/udacity_key.rsa.pem grader@18.194.31.70 -p 22.70 -p 2200 

the Key is in the notes to instructor !

<h1>STEPS ! </h1>
Summary Created Amazon Lightsail instance
Click OS only 
Choose Ubuntu

configure the ports Amazon Lightsail will allow. By default the firewall is set to only allow connects from port 22 and port 80. we made ssh listen to 2200 and NTP to (port 123)


Connect via ssh
ssh -i ~/.ssh/udacity_key.rsa.pem grader@18.194.31.70 -p 2200 

switch to root by sudo su -
add new user as grader by sudo adduser grader
the password is in the note to instructor 

give grader superuser permissions by sudo nano /etc/sudoers.d/grader
and then add grader ALL=(ALL:ALL)

then update the packages 

sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
create an SSH Key for our new user grader. From a new terminal run the command: $ ssh-keygen -f ~/.ssh/udacity.rsa
$ cat ~/.ssh/udacity.rsa.pub
Back in the server terminal locate the folder for the user grader, it should be /home/grader. Run the command $ cd /home/grader to move to the folder.
Create a directory called .ssh with the command $ mkdir .ssh
Create a file to store the public key with the command $ touch .ssh/authorized_keys
Edit that file using $ nano .ssh/authorized_keys
Now paste in the public key

change the permissions of the file and its folder by running
sudo chmod 700 /home/grader/.ssh
sudo chmod 644 /home/grader/.ssh/authorized_keys 
Change the owner of the .ssh directory from root to grader by using the command $ sudo chown -R grader:grader /home/grader/.ssh
restart its service with  sudo service ssh restart
$ ssh -i ~/.ssh/udacity.rsa grader@18.194.31.70
enforce key authentication from the ssh configuration file by editing $ sudo nano /etc/ssh/sshd_config. Find the line that says PasswordAuthentication and change it to no. Also find the line that says Port 22 and change it to Port 2200. Lastly change PermitRootLogin to no.

Restart service again sudo service ssh restart

connect again but with port 2200
configure the firewall using these commands

$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable


NOW we will go to deploy process 
First we will install the needed packages 
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git

Enable mod_wsgi with the command $ sudo a2enmod wsgi and restart Apache using $ sudo service apache2 restart.
If you input the servers IP address into a web browser you'll see the Apache2 Ubuntu Default Page
We now have to create a directory for our catalog application and make the user grader the owner

$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
then clone our application 
git clone https://github.com/Mosilhy/itemcatalo catalog

Create the .wsgi file by $ sudo nano catalog.wsgi

write this into the file

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'

rename python.py ( the main file ) to __init__.py
by mv project.py __init__.py
go   into by    cd /var/www/catalog
create virtual environment by 
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
install other packages that my project needs 
sudo apt-get install python-pip
$ sudo pip install flask
$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 #etc...

change the directory so instead of reading the file directly , go to our path to read it example
/client_secrets.json is changed to /var/www/catalog/catalog/client_secrets.json

 configure and enable our virtual host to run the site by
sudo nano /etc/apache2/sites-available/catalog.conf
<VirtualHost *:80>
    ServerName 18.194.31.70
    ServerAdmin admin@18.194.31.70
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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
    ErrorLog  /var/www/catalog/apache_errors.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


Enable to virtual host: $ sudo a2ensite catalog.conf

setting up our database :D 
sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres -i
$ psql

postgres=# CREATE USER catalog WITH PASSWORD 'password';
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
$ exit
now we will modify our project files to use postgres instead of sqllite
so we will change 
sqlite://catalog.db  to  postgresql://username:password@localhost/catalog

then we will create our database using 

sudo python database_setup.py 

then fill up some data using 
sudo python lotsofitems.py

then restart our service using 
sudo service apache2 restart

then we will go to our favourite browser to browse our ip to see our website running :D  

References:
https://github.com/callforsky/udacity-linux-configuration
https://github.com/iliketomatoes/linux_server_configuration
https://github.com/mulligan121/Udacity-Linux-Configuration
https://github.com/kongling893/Linux-Server-Configuration-UDACITY 

https://stackoverflow.com/questions/43267157/python-attributeerror-module-object-has-no-attribute-ssl-st-init

https://stackoverflow.com/questions/51283708/python-pip-package-requestsdependencywarning-when-installing-elastic-search-cura 
