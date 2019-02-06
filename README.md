
# Linux Server Configuration

#### Project Overview
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

#### Why this project?

A deep understanding of exactly what your web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.

#### What Will I Learn?

You will learn how to access, secure, and perform the initial configuration of a bare-bones Linux server. You will then learn how to install and configure a web and database server and actually host a web application.

## Server details
* IP address: `52.15.90.98`
* SSH port: `2200`
* Hostname: `ec2-52-15-90-98.us-east-2.compute.amazonaws.com`


## Configuration steps

### 1. Create a new Ubuntu Linux server instance on Amazon Lightsail
* Login into [Amazon Lightsail](https://lightsail.aws.amazon.com) using an Amazon Web Services account.
* login into the site, click Create instance.
* Choose Linux/Unix platform, OS Only and Ubuntu 18.04 LTS.
* Choose a payment plan (free for first month)
* Click the Create button to create the instance.
* The instance needs 5~10 mins to set up.
* After it is set up, you will see 'running' in the left corner of the status card.
* Write down the public IP address on a paper as you will use it a lot in the following steps.

### 2. Set up SSH key 
* From the `Account` menu on Amazon Lightsail, select `Account` and click on SSH keys tab and then download the Default Private Key.
* Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `udacity_key.pem`.

### 3. configure the ports Amazon Lightsail
By default the firewall is set to only allow connects from port 22 and port 80. We need to set up port 2200 and 123:
* Go to home page and Click the menu of status card and select `Manage`.
* Click the `Networking` tab and find the `Add another` at the bottom. Add port 123 and 2200. Amazon Lightsail allows only port 22 and 80 by default.
 

### 4. Loggin to server with SSH
* To make our key secure type `$ chmod 600 ~/.ssh/udacity_key.pem` into the terminal.
* From here we will log into the server as the user `ubuntu` with our key. From the terminal type `$ ssh -i ~/.ssh/udacity.pem ubuntu@52.15.90.98`.
* Once logged in you will see the command line change to `ubuntu@[ip-172-26-6-91]:$`
* Lets switch to the root user by typing `sudo su -` you see the command line change to `root@ip-172-26-6-91:~#`.
 
### 5. create a user called `grader`
* From the command line type `$ sudo adduser grader`. It will ask for 2 passwords and then a few other fields which you can leave blank.
* Create a file to give the user `grader` superuser privileges. To do this type `$ sudo nano /etc/sudoers.d/grader`. This will create a new file that will be the superuser configuration for grader. When nano opens type `grader ALL=(ALL:ALL) ALL`, to save the file hit Ctrl-X on your keyboard, type 'Y' to save, and return to save the filename.
* In order to prevent the `$ sudo: unable to resolve host error`, edit the hosts by `$ sudo nano /etc/hosts`, and then add `127.0.1.1 ip-10-20-37-65` under `127.0.1.1:localhost`.


### 6. Update and upgrade installed packages
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
```

### 7. create an SSH Key for new user `grader`
* On your local machine open a new Terminal window (Command+N) and input `$ ssh-keygen -f ~/.ssh/udacity.rsa`
* In the same terminal we need to read and copy the public key using the command: `$ cat ~/.ssh/udacity.rsa.pub`. Copy the key from the terminal.
* Back in the server terminal locate the folder for the user grader, it should be /home/grader. Run the command `$ cd /home/grader` to move to the folder.
* Create a directory called .ssh with the command `$ mkdir .ssh` 
* Create a file to store the public key with the command `$ touch ssh/authorized_keys`.
* Edit that file using `$ nano .ssh/authorized_keys`. 
* Now paste in the public key.
* Change the permissions of the file and its folder by running:
```
$ sudo chmod 700 /home/grader/.ssh
$ sudo chmod 644 /home/grader/.ssh/authorized_keys 
```
* Change the owner of the .ssh directory from root to grader by using the command `$ sudo chown -R grader:grader /home/grader/.ssh`.
* restart its service with `$ sudo service ssh restart`.
* Disconnect from the server.

### 8. Login to server with the `grader` account using ssh 
* From your local terminal type `$ ssh -i ~/.ssh/udacity.rsa grader@52.15.90.98`
* You should now be logged into your server via SSH.

### 9. Enforce key-based authentication & Change the SSH port from 22 to 2200
* The ssh configuration file by editing `$ sudo nano /etc/ssh/sshd_config`
* Find the line that says **PasswordAuthentication** and change it to no .
* Find the line that says **Port 22** and change it to **Port 2200**.
* Change **PermitRootLogin** to no.
* **Restart** ssh again: `$ sudo service ssh restart` and **disconnect** from the server.
* And again connect to from your local terminal type `ssh -i ~/.ssh/udacity.rsa grader@52.15.90.98 -p 2200`.

### 10. Configure Firewall (UFW)
```
$ sudo ufw allow 2200/tcp 
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable
```
* Running $ sudo ufw status should show all of the allowed ports with the firewall configuration.

### 11. Install Apache application and wsgi module
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git
```
* Enable mod_wsgi with the command `$ sudo a2enmod wsgi` and restart Apache using `sudo service apache2 restart`

### 12. Catalog application and make the user `grader` the owner
```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
```
### 13. Create the .wsgi file
* Go to this folder `$ cd var/www/catalog/`
* Create the .wsgi file by `$ sudo nano catalog.wsgi` and make sure your secret key matches with your project secret key
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
### 14. Create our virtual environment
* make sure you are in `/var/www/catalog`.
```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
```
* This is what your command line `(venv) grader@ip-52.15.90.98:/var/www/catalog$`

### 15. Cloning our Catalog Application repository
* Our application which will sit inside directory called catalog `/var/www/catalog/catalog`.
* make sure you are in `/var/www/catalog`.
* `$ git clone https://github.com/mohrabea/item_catalog.git catalog`
* Rename `application.py` in our `catalog` application folder to `__init__.py` by `$ mv project.py __init__.py`

### 16. Install all packages required for our Flask application
```
$ sudo apt-get install python-pip
$ sudo pip install flask
$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 #etc...
```
### 17. Configure and enable our virtual host to run the site
* `$ sudo nano /etc/apache2/sites-available/catalog.conf`
Paste in the following:
```
<VirtualHost *:80>
    ServerName 52.15.90.98
    ServerAlias ec2-52-15-90-98.us-east-2.compute.amazonaws.com
    ServerAdmin admin@52.15.90.98
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
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* If you need help finding your servers hostname go [here](https://whatismyipaddress.com/ip-hostname) and paste the IP address. Save and quit nano
* Enable the new virtual `host: $ sudo a2ensite catalog`.
* DISABLE the default host `$ a2dissite 000-default.conf`

### 18. Setting up the database
```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres -i
$ psql
```

### 19. Create a database user and password
```
postgres=# CREATE USER catalog WITH PASSWORD 'catalog';
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
$ exit
```
* Use nano again to edit your __init__.py, database_setup.py, and database_seeder.py files to change the database engine from `sqlite://catalog.db` to `postgresql://catalog:catalog@localhost/catalog` 



**Restart your apache server `$ sudo service apache2 restart` and now your IP address and hostname should both load your application.**



## Reference

* [https://github.com/mulligan121/Udacity-Linux-Configuration](https://github.com/mulligan121/Udacity-Linux-Configuration/blob/master/README.md)
* [https://github.com/chuanqin3/udacity-linux-configuration](https://github.com/chuanqin3/udacity-linux-configuration)
* [https://github.com/iliketomatoes/linux_server_configuration](https://github.com/iliketomatoes/linux_server_configuration)

### Authors

* **Mohammed Rabea** - *Email: moh.rabea@gmail.com* 
