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

live website on:  <strike>34.210.46.54</strike><br>
<i>Link closed as my project is done</i>
<br>

<b><big>****  Requirements  ****</big></b><br>
To run this project you should have the following things on your computer.<br>

- Any Browser.
- Internet.
- terminal for connecting to server and Configuring it.
<br>

<b><big>****  How to execute the project  ****</big></b><br>

<br><b><i>----  Connecting to the server in terminal  ----</i></b><br>
- Paste the content given in the notes in a key.pub file.
- `ssh grader@34.210.46.54 -p 2200 -i key.pub`

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
-  `sudo mkdir .ssh` .
-  `sudo touch .ssh/authorized_keys` .
- `sudo nano .ssh/authorized_keys`.
- Now copy the contents of public key generated on your local machine which you must save to `~/.ssh/` folder where '~' is your default directory and paste it in authorized_keys file..
- `sudo chmod 700 .ssh`.
- `sudo chmod 644 .ssh/authorized_keys`.
- `sudo service ssh restart` --> For restarting the ssh.
- Now, to login through the public key<br>
    `ssh -i [privateKeyFilename] grader@34.210.46.54 -p 2200`.


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
- `sudo apt-get install python-dev`.
- `sudo apt-get install python-pip`.
- `sudo pip install virtualenv`.
- Move to project folder by `cd /var/www/project`.
- `source venv/bin/activate` will activate the Virtual Environment.
- `sudo chmod -R 777 venv` to change the permissions.
- `sudo pip install flask`.
- `sudo pip install oauthclient`.
- `sudo pip install sqlalchemy`.
- `sudo pip install pyscopg2`.
- `sudo pip install request`.
- `sudo pip install httplib2`.
- `deactivate` to deactivate the Virtual Environment.

<br>
<b><i>----  Cloning Catalog project  ----</i></b><br>

- `cd /var/www`.
- `sudo mkdir project`.
- `cd project`.
- `git clone https://github.com/shubham-shar/catalog.git catalog` .
- `sudo nano catalog.wsgi` and write the following in it: <br>
```
activate_this = '/var/www/project/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/project/")

from catalog.finalproject import app as application
application.secret_key = "Add any key as you want"
```
<br>
<b>NOTE-- <b> `key is not written due to security reasons`

<br><b><i>----  Configure and Enable a new virtual host  ----</i></b><br>

- `sudo nano /etc/apache2/sites-available/catalog.conf`.
- Enter the following:
```
  <VirtualHost *:80>
  ServerName 34.210.46.54
  ServerAlias ec2-34-210-46-54.us-west-2.compute.amazonaws.com
  ServerAdmin admin@34.210.46.54
  WSGIDaemonProcess catalog python-path=/var/www/project:/var/www/project/venv/lib/python2.7/site-packages
  WSGIProcessGroup catalog
  WSGIScriptAlias / /var/www/project/catalog.wsgi
  <Directory /var/www/project/catalog/>
      Order allow,deny
      Allow from all
  </Directory>
  Alias /static /var/www/project/catalog/static
  <Directory /var/www/project/catalog/static/>
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

<b><big>****  References  ****</big></b>
- Stackoverflow website.
- Amazon web services documentation.
- Flask documentation on how to deploy a Flask.
<br>

<b><big>****  Thank you  ****</big></b><br>
