<b><big>****  This document contains following  ****</big></b><br>
<ol>
<li> Project-Description.</li>
<li> Requirements.</li>
<li> How to execute the project.</li>
</ol>
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
<ol>
<li>Any Browser</li>
<li>Internet</li>
</ol>
<br>

<b><big>****  How to execute the project  ****</big></b><br>
<b><i>----  Updating the softwares  ----</i></b><br>
<ol>
<li> `sudo apt-get update`</li>
<li> `sudo apt-get upgrade`</li>
</ol>

<b><i>----  Creating user grader and giving sudo access ----</i></b><br>
<ol>
<li> `sudo adduser grader`</li>
<li> `sudo touch /etc/sudoers.d/grader` </li>
<li> `sudo nano /etc/sudoers.d/grader` </li>
<li>Now, In this file type <br> `grader ALL=(ALL) NOPASSWD:ALL`</li>
</ol>

<b><i>----  Allowing grader user to login by public key  ----</i></b><br>
<ol>
<li>When connected as a root user to server type <br>`su - grader`</li>
<li>`mkdir .ssh`</li>
<li>`touch .ssh/authorized_keys`</li>
<li>`nano .ssh/authorized_keys`</li>
<li>Now copy the contents of public key generated on your local machine which you must save to `~/.ssh/` folder where '~' is your default directory and paste it in authorized_keys file.</li>
<li>`chmod 700 .ssh`</li>
<li>`chmod 644 .ssh/authorized_keys`</li>
<li>`service ssh restart` --> For restarting the ssh</li>
<li>Now, to login through the public key<br>
    `ssh -i [privateKeyFilename] grader@34.208.109.124`</li>
</ol>

<b><i>---- Disabling Root access and password login  ----</i></b><br>
	In the the `/etc/ssh/sshd_config` file change the following:
<li>`PermitRootLogin` to `PermitRootLogin no`</li>
<li>`PasswordAuthentication yes` to `PasswordAuthentication no`</li>

<b><i>----  Changing default port from 22 to 2200  ----</i></b><br>
<ol>
<li>`sudo nano /etc/.ssh/sshd_config`</li>
<li>Change the line `Port 22` to `Port 2200`</li>
</ol>

<b><i>---- Configuring Firewall to allow certain ports  ----</i></b><br>
<ol>
<li>Check the Firewall status by typing<br>`sudo ufw status`</li>
<li>If its inactive then proceed without executing the next command, else execute the next command</li>
<li>`sudo ufw disable` </li>
<li>`sudo ufw default allow incoming`</li>
<li>`sudo ufw allow 2200/tcp`</li>
<li>`sudo ufw allow 80/tcp`</li>
<li>`sudo ufw allow 123/udp`</li>
<li>`sudo ufw enable`</li>
</ol>

<b><i>----  Changing the local timezone to UTC  ----</i></b><br>
<ol>Type the following to set timezone to UTC:
<li>`sudo  timedatectl set-timezone Etc/UTC`</li>
</ol>

<b><i>----  Installing Apache, mod_wsgi and PostgreSQL  ----</i></b><br>
<ol>
<li>`sudo apt-get install apache2`</li>
<li>`sudo apt-get install libapache2-mod-wsgi`</li>
<li>`sudo apache2ctl restart`</li>
<li>`sudo apt-get install postgresql`</li>
</ol>

<b><i>----  Installing Additional Pakages and Creating .wsgi file  ----</i></b><br>
<ol>
<li>`Sudo apt-get install git`</li>
<li>`sudo apt-get install python-pip`</li>
<li>`sudo pip install flask`</li>
<li>`sudo pip install oauthclient`</li>
<li>`sudo pip install sqlalchemy`</li>
<li>`sudo pip install pyscopg2`</li>
<li>`sudo apt-get install python-dev`</li>
</ol>

<b><i>----  Cloning Catalog project  ----</i></b><br>
<ol>
<li>`cd /var/www`</li>
<li>`sudo mkdir catalog`</li>
<li>`sudo chown -R grader:grader catalog`</li>
<li>`cd catalog`</li>
<li>`git clone https://github.com/shubham-shar/catalog.git catalog` </li>
<li>`sudo nano catalog.wsgi` and write the following in it: <br>
`#!/usr/bin/python
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0, "/var/www/catalog/")
 from catalog import finalproject as application`</li>
</ol>

<b><i>----  Configure and Enable a new virtual host  ----</i></b><br>
<ol>
<li>`sudo nano /etc/apache2/sites-available/catalog.conf`</li>
<li>Enter the following: <br>
`<VirtualHost *:80>
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
</VirtualHost>`</li>
<li>`sudo a2ensite catalog`</li>
</ol>

<b><i>----  Configuring PostgreSQL  ----</i></b><br>
<ol>
<li>`sudo su - postgres`</li>
<li>`psql`</li>
<li>`CREATE DATABASE catalog;`</li>
<li>`CREATE USER catalog;`</li>
<li>`ALTER USER catalog WITH PASSWORD 'apppass';`</li>
<li>`GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`</li>
<li>`\q`</li>
<li>`exit`</li>
<li>Find and replace the line with`engine = create_engine` with the following: <br>
		`engine = create_engine('postgresql://catalog:apppass@localhost/catalog')`</li>
<li>`sudo python database_setup.py`</li>
	<li>`sudo service apache2 restart`</li>
</ol>
That's it, now enjoy the webpage.<br>

<b><big>****  Thank you  ****</big></b><br>
