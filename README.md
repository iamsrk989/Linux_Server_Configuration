# Linux_Server_Configuration
Project for Udacity Full Stack Web Developer Nanodegree

In this project a new Linux Server(ubuntu 16.04 LTS) is taken and necessary packages are added. Finally Project item catalog is displayed as a WSGI app.

The project is currently live at http://34.205.142.105

To Access the Server log into ssh -> ssh -i <privatekey> grader@34.205.142.105


Summary of configurations made:

1. An Ubuntu instance was created in Amazon lightsail.
2. After logging into the SSH all the packages are checked for updates and updated.

sudo apt-get update
sudo apt-get upgrade

3. User grader is added

sudo adduser grader

4. User grader is given sudo access

sudo touch /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader

5. Now we change to user grader

su - grader

6. We then generate keys using ssh-keygen command in local machine and store it securely.

7.Back in the server we make a directory .ssh and a file authorized_keys inside that directory

mkdir .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys
(We paste the contents of the public key file inside here)

8. We change the permissions of the directory and the file.

chmod 700 .ssh
chmod 644 .ssh/authorized_keys

9.Exit ssh window and relogin from terminal

10. Next change the SSH port from 22 to 2200

sudo nano /etc/ssh/sshd_config
(Change the line with port 22 to port 2200)

11. Reload SSH

sudo service ssh restart

12. Configuring the Uncomplicated Firewall

sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable

13. Configure Time to UTC

sudo dpkg-reconfigure tzdata
(Choose place as none of the above and Timezone as UTC)

14. Install Apache, Mod-wsgi

sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi

15. Restart Apache

sudo service apache2 restart

16. Install Postgresql

sudo apt-get postgresql
psql --version

17. Login to postgresql to configure

sudo su - postgres

18. Enter postgresql shell

psql

19. Create New Database and Username Privilages

CREATE DATABASE catalog;
CREATE USER catalog;
ALTER ROLE catalog WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
\q

20. The exit user postgres

21. Check if remote connection allowed in postgresql

sudo nano /etc/postgresql/9.5/main/pg_hba.conf
(if allowed disable remote connection to database)

22. Install git

sudo apt-get install git

23. Move to www folder

cd /var/www

24. Make folder FlaskApp

sudo mkdir FlaskApp

25. Move into FlaskApp

cd FlaskApp

26. Clone Project Item catalog into the folder

sudo git clone https://github.com/iamsrk989/item_catalog.git
sudo mv ./item_catalog ./FlaskApp

27. move into FlaskApp folder now th folder structure will look like (/var/www/FlaskApp/FlaskApp)

cd FlaskApp

28. Rename project.py into __init__.py

sudo mv project.py __init__.py

29. CHange references of sqlite to postgresql in all the python files.

30. Install pip , psycopg2, flask, sqlalchemy

sudo apt-get install python-pip
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy

31. The run database_setup.py

sudo python database_setup.py

32. Create FlaskApp.conf and edit

sudo nano /etc/apache2/sites-available/FlaskApp.conf
(add following code and save it)
<VirtualHost *:80>
	ServerName 52.24.125.52
	ServerAdmin qiaowei8993@gmail.com
	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/static
	<Directory /var/www/FlaskApp/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

33. Enable virtual host

sudo a2ensite FlaskApp

34. Create flaskapp.wsgi File in /var/www/FlaskApp

cd ..
sudo nano flaskapp.wsgi
(Add following code and save)
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'

35. In __init__.py

Change the path of 'client_secrets.json' to r'/var/www/FlaskApp/FlaskApp/client_secrets.json
Change the path of 'fb_client_secrets.json' to r'/var/www/FlaskApp/FlaskApp/fb_client_secrets.json

36. Restart Apache

sudo service apache2 restart



Reference:

1.http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt
2.http://packages.ubuntu.com/
3.https://www.linode.com/docs/websites/apache/apache-and-modwsgi-on-ubuntu-14-04-precise-pangolin
4.https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04
5.https://www.postgresql.org/docs/9.0/static/sql-alterrole.html
6.https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
