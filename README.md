# Linux Server Configuration

Project url: http://my-list.club
Project IP: http://178.128.12.115/


I have deployed the Item Catalog project to a linux server (Digital Ocean).

Github link to the Item catalog Project: https://github.com/MahmoudSoliman922/Item-list.git

## Table of Contents

1. Start a new server on Digital Ocean.
2. SSH into the server.
3. Update all currently installed packages.
4. Change the SSH port from 22 to 2200.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
6. Create a new user account named `grader`.
7. Give `grader` the permission to sudo.
8. Create an SSH key pair for `grader` using the ssh-keygen tool.
9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.
11. Install git.
12. Clone and setup your git project.
13. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser.
14. Disable root remote login and enforce key-based authentication.

## Step 1:

To start a new server on Digital Ocean, create a new droplet. Use your prefered specifications -used the minimum specifications-. 


## Step 2:

SSh into the server by typing:
```sh
ssh root@178.128.12.115 -p 22
```
## Step 3:

Update currently installed packages, run the following command-

```sh
sudo apt-get update
sudo apt-get upgrade
```


## Step 4:

Change ssh port to 2200 by doing the following :

```sh
sudo nano /etc/ssh/sshd_config
```

Locate "Port 22" in that file. Change it to "Port 2200".

Restart ssh service.

```sh
sudo service ssh restart
```


## Step 5:

To configure UFW to allow given connections, follow the steps:

```sh
sudo ufw allow 2200
sudo ufw allow www
sudo ufw allow ntp
```

Then enable ufw.

```sh
sudo ufw enable
```

Now ssh port has been changed to 2200. Try exiting the ssh connection and re-connecting with the following command.

```sh
ssh -p 2200 root@178.128.12.115
```

## Step 6:

To create a user called `grader`, run the following command.

```sh
sudo adduser grader
```

Fill in the details, use `1234554321` as the password for this user.


## Step 7:

To give `grader` sudo permission, we first create `grader` file inside `sudoers.d`.

```sh
touch /etc/sudoers.d/grader
```

Then we open the file in `nano` (`sudo nano /etc/sudoers.d/grader`) and add the following-

```sh
grader ALL=(ALL) NOPASSWD:ALL
```


## Step 8:

We will now setup a ssh key-pair for grader.

1- 1st in my local machine I'll run this command:
```sh
ssh-keygen
```
2- Then will provide it with the required path 
```sh
(\home\.ssh\Auth_Key) #I gave it password ("1234554321")
```
3- Open the Auth_Key.pub
4- copy the contents. (contents are provided in the instructor notes)
5- And then on the grader account run the following commands:
```sh
mkdir .ssh
chmod 700 .ssh
nano .ssh/authorized_keys
#paste the content of Auth_Key.pub here
chmod 644 .ssh/authorized_keys
```


Restart the ssh service.

```sh
sudo service ssh restart
```

Now you should be able to login as `grader`. Exit current connection and do the following.

```sh
ssh -p 2200 -i ~/.ssh/Auth_Key grader@178.128.12.115
```


## Step 9:

To configure timezone, run the following command.

```sh
sudo dpkg-reconfigure tzdata
```

Select "None of the above" and then select UTC.


## Step 10:

Start apache service.

```sh
sudo service apache2 restart
```



## Step 11:

To install git, run-

```sh
sudo apt-get install git
```


## Step 12:

Our project is at https://github.com/MahmoudSoliman922/Item-list.git

First we need to clone it on server.

```sh
cd /var/www/html
sudo git clone https://github.com/MahmoudSoliman922/Item-list.git
```

## Step 13:

Now we need to run the project using Apache and mod-wsgi. So we will first create a configuration file for your project.

```sh
sudo nano /etc/apache2/sites-available/FlaskApp.conf
```

It should have the following components.

```xml
<VirtualHost *:80>
                ServerName 178.128.12.115
                ServerAlias my-list.club
                ServerAdmin admin@mywebsite.com
                WSGIScriptAlias / /var/www/html/FlaskApp/flaskapp.wsgi
                <Directory /var/www/html/FlaskApp/FlaskApp/Item-list/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/html/FlaskApp/FlaskApp/Item-list/static
                <Directory /var/www/html/FlaskApp/FlaskApp/Item-list/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

and also edit this file for wsgi configurations:
```sh
sudo nano /var/www/html/FlaskApp/flaskapp.wsgi
```
to be like this:

```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/html/FlaskApp/FlaskApp/Item-list/")

from index import app as application
application.secret_key = 'Add your secret key'

```

Once this is done, enable the site and restart Apache.

```sh
sudo a2ensite flaskapp  # enable site
sudo service apache2 reload
```

The server should be live now. Visit the IP to check (http://my-list.club). If an error occurs, check the logs.

```sh
sudo cat /var/log/apache2/error.log
```

## Step 14:

To disable root login & password-based login through ssh, open the ssh config file.

```sh
sudo nano /etc/ssh/sshd_config
```

Make changes as shown below.

```sh
PermitRootLogin no
PasswordAuthentication no
```

Save the file and restart ssh server.

```sh
sudo service ssh restart
```

## References:
1- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

2- https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-14-04

3- https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars

4- https://docs.nexcess.net/article/how-to-verify-dns-propagation.html

5- https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server

6- https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04

## System packages:
```sh
python , apache2 , mod_wsgi 
```
in addition to the required softwares for the Item-catalog project.