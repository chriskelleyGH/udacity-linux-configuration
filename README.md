
# Linux Server Configuration


A README file is included in the GitHub repo containing the following information: IP address, URL, summary of software installed, summary of configurations made, and a list of third-party resources used to complete this project.

### Server Information

IP Address: 
URL: 

### Software Installed



### Configuration 

Update all installed packages
```sh
sudo apt-get update
sudo apt-get upgrade
```

Add the following to the Lightsail management console: Application = Custom, Protocol = TCP, Port = 2200

add to lightsail management console: Application = Custom, Protocol = TCP, Port = 2200
edit the port in sshd_config in /etc/ssh/ folder to 2200.  uncomment # port 22
service ssh restart
download private key from amazon

change permissions of the key.   This step is required.
$ chmod 600 LightsailDefaultKey-us-east-1.pem
run ssh -i [fileName] [username]@[Public IP]
$ ssh -i LightsailDefaultKey-us-east-1.pem -p 2200 ubuntu@18.233.10.243

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

```sh
$ sudo ufw status
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow http
$ sudo ufw allow ntp
$ sudo ufw enable
$ sudo ufw status
```

Create a new user account named grader.

```sh
$ sudo adduser grader
```

Give grader sudo permission.  Create grader file inside sudoers.d.

```sh
touch /etc/sudoers.d/grader
```

Open the file in nano (sudo nano /etc/sudoers.d/grader) and add the following-

```sh
grader ALL=(ALL) NOPASSWD:ALL
```

Create an SSH key pair for grader using the ssh-keygen tool.

Configure the local timezone to UTC.

```sh
sudo dpkg-reconfigure tzdata
```

Install and configure Apache to serve a Python mod_wsgi application.

```sh
Install Apache sudo apt-get install apache2
Install mod_wsgi sudo apt-get install python-setuptools libapache2-mod-wsgi
Restart Apache sudo service apache2 restart
```

Install and configure PostgreSQL

```sh
sudo apt-get install postgresql
```

Check if no remote connections are allowed

```sh
sudo nano /etc/postgresql/10/main/pg_hba.conf
```

Login as user "postgres" 

```sh
sudo su - postgres
```

Get into postgreSQL shell 

```sh
psql
```

Create a new database named catalog and create a new user named catalog in postgreSQL shell

```sh
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
```

Set a password for user catalog

```sh
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
```

Give user "catalog" permission to "catalog" application database

```sh
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```

Quit postgreSQL 

```sh
postgres=# \q
```

Exit from user "postgres"

```sh
exit
```

Install Git 

```sh
sudo apt-get install git
```

Create the application directory 
```sh
cd /var/www
sudo mkdir FlaskApp
cd FlaskApp
```

Clone the Catalog App to the server

```sh
sudo git clone https://github.com/chriskelleyGH/catalog_app.git
```

Rename the project's name

```sh
 sudo mv ./catalog_app ./FlaskApp
 ```
 
Rename application.py to __init__.py

```sh
cd FlaskApp
sudo mv application.py __init__.py
```

Update 2 lines of code in __init__.py to refer to the new path of clinet json file

```sh
sudo nano __init__.
```

First change:

```sh
CLIENT_ID = json.loads(open('/var/www/FlaskApp/FlaskApp/client_secrets.json', 'r').read())['web']['client_id']
```

Second change:

```sh
oauth_flow = flow_from_clientsecrets('/var/www/FlaskApp/FlaskApp/client_secrets.json', scope=‘')
```

Edit database_setup.py, __init__.py,data.py, and any other reference to sqlite in a file and change 

```sh
engine = create_engine('sqlite:///catalog.db') to
engine = create_engine('postgresql://catalog:udacity2019@localhost/catalog')
```

Install pip 

```sh
sudo apt-get install python-pip
```

Use pip to install dependencies (MAKE SURE TO CREATE REQUIREMENTS.TXT FILE)

```sh
$ sudo pip install -r requirements.txt
```

Install psycopg2
```sh
sudo apt-get -qqy install postgresql python-psycopg2
```

Create database schema

```sh
sudo python database_setup.py
```

Configure and Enable a New Virtual Host

Create FlaskApp.conf to edit

```sh
sudo nano /etc/apache2/sites-available/FlaskApp.conf
```

Add the following lines of code to the file to configure the virtual host.

```sh
<VirtualHost *:80>
    ServerName 18.233.10.243
    ServerAlias 18.233.10.243.xip.io
    ServerAdmin ubuntu@18.233.10.243
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
```
       
Enable the virtual host with the following command

```sh
sudo a2ensite FlaskApp
```

Create the .wsgi File under /var/www/FlaskApp:

```sh
cd /var/www/FlaskApp
sudo nano flaskapp.wsgi
```

Add the following lines of code to the flaskapp.wsgi file:

```sh
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")
from FlaskApp import app as application
application.secret_key = ’super_secret_key'
```

Restart Apache

```sh
sudo service apache2 reload
sudo service apache2 restart
```

Configure Google OAuth

Visit https://console.developers.google.com/apis and obtain Oauth credentials
Create a new project
Choose credentials from the menu on the left
Create an OAuth Client ID
Configure the consent screen
When you're presented with a list of application types, choose Web application.  Name project item catalog
Enter authorized javascript origins
Enter authorized redirect URI

Authorized Javascript Origins
http://18.233.10.243.xip.io

Authorized Redirect URIS
http://18.233.10.243.xip.io/login
http://18.233.10.243.xip.io/gconnect

In the Auth Content Screen tab. add xip.io in the Authorized Domains
       
Update login.html file with google client ID

```sh
          <div id="signinButton">
          <span class="g-signin"
            data-scope="openid email"
            data-clientid="YOUR_CLIENT_ID_GOES_HERE.apps.googleusercontent.com"
            data-redirecturi="postmessage"
            data-accesstype="offline"
            data-cookiepolicy="single_host_origin"
            data-callback="signInCallback"
            data-approvalprompt="force">
          </span>
        </div>
```

From google developer console, go to project and download JSON.  (Refer To Lesson 6-9 If Necessary) 

Rename the file client_secrets.json

Replace the client_secrets.json file (from the developer item catalog application) on the LightSail server.   This will be in the same directory as the main python file.

Make sure .git directory is not available from the browser.

Put this in an .htaccess file at the root of your web server:

```sh
RedirectMatch 404 /\.git
```

### References

The following resources were refernced to complete this project.

https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e

