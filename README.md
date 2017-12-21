# Linux Server Configuration
* This project included access, secure, and perform the initial configuration of a bare-bones Linux server, it will also include how to install and configure a web and database server and actually host a web application.

## Sever Details
* Public IP : [104.131.87.6](104.131.87.6)
* Server OS : Ubuntu 16.04 x64
* SSH Port: 2200
* Server Provider: [DigitalOcean](https://m.do.co/c/c1a77ba6d4e1)
* Web Apllication: [Item-Catalog-Application](https://github.com/arrickx/Item-Catalog-Application)
* URL: [104.131.87.6](104.131.87.6)

### Summary of Software/Package Installed
- apache2
- libapache2-mod-wsgi
- postgresql
- git
- pip
- virtualenv
- flask
- sqlalchemy
- google-auth
- python

### Setting Up the Server

##### 1. Create a new server
* Start a new Ubuntu Linux server on DigitalOcean.
* Use DigitalOcean to access console(login as root)

##### 2. Initial setup and secure  the server
* Set the Timezone to UTC using the following code
```sh
~ $ sudo timedatectl set-timezone UTC
```
* And use this to verify the status
```sh
~ $ sudo timedatectl status
```
##### 3. Change the SSH Port from 22 to 2200
* Open the SSH configuration
```sh
~ $ sudo nano /etc/ssh/sshd_config
```
* Change Port to **2200**
* Save and restart the ssh
```sh
~ $ sudo service ssh restart
```
##### 4. Configure the Firewall (UFW)
```sh
~ $ sudo ufw status #check status

~ $ sudo ufw default deny incoming #deny ALL incoming
~ $ sudo ufw default allow outgoing #allow ALL outgoing

~ $ sudo ufw allow 2200/tcp #allow SSH(remember to change SSH port before do this step)
~ $ sudo ufw allow 80/tcp #allow HTTP
~ $ sudo ufw allow 123/udp #allow NTP

~ $ sudo ufw enable # turn on the firewall(before this step firewall is still inactive/turn off)
~ $ sudo ufw status #check status to make sure it works
```

##### 5. Create a new user
* Create a new user called grader.
```sh
~ $ sudo adduser grader
```
* Give grader the permission to perform sudo commands(without password).
```sh
~ $ sudo visudo #open /etc/sudoers files
#find the following line
root    ALL=(ALL:ALL) ALL
#and add the following line to give the grader sudo permission
root    ALL=(ALL:ALL) NOPASSWD:ALL
```
* Setup an SSH key for the grader by using the **ssh-keygen**.
```sh
#Things to do on the local machine

type the following code on local machine's terminal to generate ssh key.
~ $ ssh-keygen
change the files path and name to ~/.ssh/grader
read the public key with the following code
~ $ cat ~/.ssh/grader.pub

#Things to do on the remote machine/console
~ $ su - grader #switch to user grader
~ $ nano ~/.ssh/authorized_keys #create file and folder for authorize key.

#Now go back to the local machine and copy the public key and paste it on the  authorized_keys file.

~ $ chmod 700 ~/.ssh && chmod 600 ~/.ssh/* #set the permission for those folder and files
~ $ sudo nano /etc/ssh/sshd_config #let key-based authentication the only way to login
change the line PasswordAuthentication yes to no
remember to save it
```
##### 6. Use SSH login as grader
* Use following code on the local machine to login as grader
```sh
~ $ ssh grader@104.131.87.6 -p 2200 -i ~/.ssh/grader
```
##### 7. Install Apache and mod_wsgi
* after login as grader, use the following code to install apache
```sh
~ $ sudo apt-get install apache2
```
* at the time finished install apache, we can vist our ip number to check if it works. if the default page was show, it works correctly
* then we could install mod_wsgi(depends on web-app built-n python version)
```sh
~ $ sudo apt-get install libapache2-mod-wsgi #only for python2
```
##### 8. Install and setup PostgreSQL
* psql is our database server, so we need to install it first
```sh
~ $ sudo apt-get install postgresql
```
* psql is default disable the remote connections, but we still need to make sure it by visiting it's configure file
```sh
~ $ sudo nano /etc/postgresql/9.5/main/pg_hba.conf #9.5 is the version number
```
* we need to ake sure that it only allows connections from localhost (either 127.0.0.1 for IPv4 or ::1 for IPv6) and no external connections.
* after making sure everything works correctly we will need to setup the psql for further usage.
```sh
~ $ sudo -u postgres psql #use this code to get into psql prompt
then create a database with username and password for security

CREATE ROLE catalog WITH LOGIN PASSWORD 'topsecret'; # catalog is user and topsecret is password

then create a new database called catalogdb

CREATE DATABASE catalogdb;

use \q to exit psql prompt
```
##### 8. Tweak our Catalog Application
* our catalog application was built originally with sqlite for easy access, but now we are moving to psql. so we will need to do that first
```sh
in our application, we will need to change all of
create_engine('sqlite:///catalogs.db') to
create_engine("postgresql://catalog:topsecret@localhost/catalogdb")
```
* then we are using built in python read files function on the application, but in ubuntu it might not work as good as it suppose to be, so we will need to change it as well.
```sh
we will need to change client_secrets.json to static location
/var/www/itemcatalog/itemcatalog/client_secrets.json
```
* we also need to move **app.secret_key** outside of **if __name__ == '__main__':**
* last but not least, we will need to include all python packages for the virtual machine, so we will also include a **requirement.txt** files in the application.
* after tweak the catalog application, I create another branch called **online** and push it to **GitHub**. so now i have the same working catalog application that works well on both local machine and a server.
* to make sure google login can runs correctly, we will also need to add our **ip address** to the **Authorized JavaScript origins** of our **Google’s Developer Console**.

##### 9. Deploy the Item Catalog Application
* now it's time to make all things together
* first we will need to clone our **online branch** from **GitHub** to the correct folder
```sh
a. we will need to go to the correct directory
~ $ cd /var/www/itemcatalog/

b. clone the correct branch to the correct folder
~ $ git clone --single-branch https://github.com/arrickx/Item-Catalog-Application --branch online itemcatalog
now the application is located inside  /var/www/itemcatalog/itemcatalog
```
* Set up the virtual machine
```sh
a. install pip in order to install virtualenv
~ $ sudo apt-get install python-pip

b. use pip to install virtualenv
~ $ sudo pip install virtualenv

c. make sure we are on the right folder
~ $ cd /var/www/itemcatalog/itemcatalog/

d. initiate our virtual environment and select the correct python version
~ $ virtualenv -p python venv

e. activate the virtual environment
~ $ source venv/bin/activate

f. install the requirements python packages
~ $ pip install -r requirements.txt

g. initiate our database
~ $ python lotsofitems.py
~ $ python project.py #after intial the server use ctrl+C to get out

h. Deactivate our virtual enviroment
~ $ deactivate
```
* now we will need to create the wsgi file
```sh
~ $ sudo nano /var/www/itemcatalog/itemcatalog.wsgi
```
* and we will need to add the following content inside
> import sys
sys.path.insert(0, '/var/www/itemcatalog/itemcatalog')
#activate_this is for activate the packages for virtual environment
activate_this = '/var/www/itemcatalog/itemcatalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))
from project import app as application
* create a new configuration file for our applications virtual host by running
```sh
~ $ sudo nano /etc/apache2/sites-available/itemcatalog.conf
```
* and add the following content inside

```sh
<VirtualHost *:80>
  ServerName ubuntu-512mb-nyc3-01
  ServerAlias 104.131.87.6
  ServerAdmin local@local
  WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi
  <Directory /var/www/itemcatalog/itemcatalog/>
      Order allow,deny
      Allow from all
  </Directory>
 Alias /static /var/www/itemcatalog/itemcatalog/static
  <Directory /var/www/itemcatalog/itemcatalog/static/>
      Order allow,deny
      Allow from all
  </Directory>
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Enable our virtual host.
```sh
~ $ sudo a2ensite itemcatalog
```
* Restart Apache
```sh
~ $ sudo apache2ctl restart
```
##### 10. Finish
* try our [IP address](104.131.87.6) again to test out our application.

### References:
* [Flask and mod_wsg](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Virtualenv](https://virtualenv.pypa.io/en/stable/userguide/#usage)
* [Digital Ocean: PostgreSQL on Ubuntu ](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
* [Digital Ocean: How To Add, Delete, and Grant Sudo Privileges to Users ](https://www.digitalocean.com/community/tutorials/how-to-add-delete-and-grant-sudo-privileges-to-users-on-a-debian-vps)
* [Digital Ocean: Initial Server Setup with Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)
* [Digital Ocean: Deploy a Flask Application on an Ubuntu ](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [pg_hba.conf file ](https://www.postgresql.org/docs/9.5/static/auth-pg-hba-conf.html)
* [mod_wsgi](http://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html)
