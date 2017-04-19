<b><big>****  This document contains following  ****</big></b><br>

- Project-Description.
- Requirements.
- How to execute the project.
<br>

<b><big>****  Project-Description  ****</big></b><br>
In this project, i have configured a linux server. The server was initiated by Amazon Web services.
The project includes making neccessary changes to make the server secure of many outside harmful attacks.
I have added a user name grader and given him all the neccessary and needed to join the server.
lastly, i have uploaded my previous project name catalog so that my server shows it online without any
error. For this we will do the steps included in <b>How to execute the project</b>.

live website on:  http://34.208.109.124
<br>

<b><big>****  Requirements  ****</big></b><br>
To run this project you should have the following things on your computer.<br>

- Any Browser.
- Internet.

<br>

<b><big>****  How to execute the project  ****</big></b><br>
<br><b><i>----  Updating the softwares  ----</i></b><br>

-  `sudo apt-get update`.
-  `sudo apt-get upgrade`.


<br><b><i>----  Creating user grader and giving sudo access ----</i></b><br>

-  `sudo adduser grader`.
-  `sudo touch /etc/sudoers.d/grader` .
-  `sudo nano /etc/sudoers.d/grader` .
- Now, In this file type <br> `grader ALL=(ALL) NOPASSWD:ALL`.


<br><b><i>----  Allowing grader user to login by public key  ----</i></b><br>

- When connected as a root user to server type <br>`su - grader`.
-  `mkdir .ssh` .
-  `touch .ssh/authorized_keys` .
- `nano .ssh/authorized_keys`.
- Now copy the contents of public key generated on your local machine which you must save to `~/.ssh/` folder where '~' is your default directory and paste it in authorized_keys file..
- `chmod 700 .ssh`.
- `chmod 644 .ssh/authorized_keys`.
- `service ssh restart` --> For restarting the ssh.
- Now, to login through the public key<br>
    `ssh -i [privateKeyFilename] grader@34.208.109.124`.


<br><b><i>---- Disabling Root access and password login  ----</i></b><br>
	In the the `/etc/ssh/sshd_config` file change the following:
- `PermitRootLogin` to `PermitRootLogin no`.
- `PasswordAuthentication yes` to `PasswordAuthentication no`.

<br><b><i>----  Changing default port from 22 to 2200  ----</i></b><br>

- `sudo nano /etc/.ssh/sshd_config`.
- Change the line `Port 22` to `Port 2200`.


<br><b><i>---- Configuring Firewall to allow certain ports  ----</i></b><br>

- Check the Firewall status by typing<br>`sudo ufw status`.
- If its inactive then proceed without executing the next command, else execute the next command.
- `sudo ufw disable` .
- `sudo ufw default allow incoming`.
- `sudo ufw allow 2200/tcp`.
- `sudo ufw allow 80/tcp`.
- `sudo ufw allow 123/udp`.
- `sudo ufw enable`.


<br><b><i>----  Changing the local timezone to UTC  ----</i></b><br>
Type the following to set timezone to UTC:
- `sudo  timedatectl set-timezone Etc/UTC`.

<br>
<b><i>----  Installing Apache, mod_wsgi and PostgreSQL  ----</i></b><br>

- `sudo apt-get install apache2`.
- `sudo apt-get install libapache2-mod-wsgi`.
- `sudo apache2ctl restart`.
- `sudo apt-get install postgresql`.

<br>
<b><i>----  Installing Additional Pakages and Creating .wsgi file  ----</i></b><br>

- `Sudo apt-get install git`.
- `sudo apt-get install python-pip`.
- `sudo pip install flask`.
- `sudo pip install oauthclient`.
- `sudo pip install sqlalchemy`.
- `sudo pip install pyscopg2`.
- `sudo apt-get install python-dev`.

<br>
<b><i>----  Cloning Catalog project  ----</i></b><br>

- `cd /var/www`.
- `sudo mkdir catalog`.
- `sudo chown -R grader:grader catalog`.
- `cd catalog`.
- `git clone https://github.com/shubham-shar/catalog.git catalog` .
- `sudo nano catalog.wsgi` and write the following in it: <br>
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import finalproject as application
```

<br><b><i>----  Configure and Enable a new virtual host  ----</i></b><br>

- `sudo nano /etc/apache2/sites-available/catalog.conf`.
- Enter the following:
```
  <VirtualHost *:80>
  ServerName 34.208.109.124
  ServerAlias ec2-34-208-109-124.us-west-2.compute.amazonaws.com
  ServerAdmin admin@34.208.109.124
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
- `sudo a2ensite catalog`.

<br>
<b><i>----  Configuring PostgreSQL  ----</i></b><br>

- `sudo su - postgres`.
- `psql`.
- `CREATE DATABASE catalog;`.
- `CREATE USER catalog;`.
- `ALTER USER catalog WITH PASSWORD 'apppass';`.
- `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`.
- `\q`.
- `exit`.
- Find and replace the line with`engine = create_engine` with the following: <br>
		`engine = create_engine('postgresql://catalog:apppass@localhost/catalog')`.
- `sudo python database_setup.py`.
- `sudo service apache2 restart`.

<br>
That's it, now enjoy the webpage.<br>

<b><big>****  Thank you  ****</big></b><br>
