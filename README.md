
# Linux Server Configuration


### Server Information

IP Address: 18.233.10.243

SSH Port: 2200

URL: http://18.233.10.243.xip.io/

### Software Installed

apache2

python-setuptools 

libapache2-mod-wsgi

PostgreSQL

bleach==3.1.0

certifi==2018.11.29

chardet==3.0.4

Click==7.0

Flask==1.0.2

Flask-HTTPAuth==3.2.4

Flask-SQLAlchemy==2.3.2

httplib2==0.12.1

idna==2.8

itsdangerous==1.1.0

Jinja2==2.10

MarkupSafe==1.1.0

oauth2client==4.1.3

packaging==19.0

passlib==1.7.1

psycopg2-binary==2.7.7

pyasn1==0.4.5

pyasn1-modules==0.2.4

pycodestyle==2.5.0

pyparsing==2.3.1

redis==3.2.0

requests==2.21.0

rsa==4.0

six==1.12.0

SQLAlchemy==1.2.18

urllib3==1.24.1

webencodings==0.5.1

Werkzeug==0.14.1

### Configuration 

1 Update all installed packages

```sh
sudo apt-get update
sudo apt-get upgrade
```

2 Add the following to the Lightsail management console: Application = Custom, Protocol = TCP, Port = 2200

3 Change the port in **/etc/ssh/sshd_config** to 2200. 

```sh
service ssh restart
```

4 Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

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

5 Create a new user account named grader.

```sh
$ sudo adduser grader
```

6 Give grader sudo permission.  Create grader file inside sudoers.d.

```sh
touch /etc/sudoers.d/grader
```

7 Open the file in nano (sudo nano /etc/sudoers.d/grader) and add the following-

```sh
grader ALL=(ALL) NOPASSWD:ALL
```

8 Create an SSH key pair for grader using the ssh-keygen tool.  Upload public key to the server.

9 Configure the local timezone to UTC.

```sh
sudo dpkg-reconfigure tzdata
```

10 Install and configure Apache to serve a Python mod_wsgi application.

```sh
Install Apache sudo apt-get install apache2
Install mod_wsgi sudo apt-get install python-setuptools libapache2-mod-wsgi
Restart Apache sudo service apache2 restart
```

11 Install and configure PostgreSQL

```sh
sudo apt-get install postgresql
```

12 Check if no remote connections are allowed

```sh
sudo nano /etc/postgresql/10/main/pg_hba.conf
```

13 Login as user "postgres" 

```sh
sudo su - postgres
```

14 Get into postgreSQL shell 

```sh
psql
```

15 Create a new database named catalog and create a new user named catalog in postgreSQL shell

```sh
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
```

16 Set a password for user catalog

```sh
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
```

17 Give user "catalog" permission to "catalog" application database

```sh
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```

18 Quit postgreSQL 

```sh
postgres=# \q
```

19 Exit from user "postgres"

```sh
exit
```

20 Install Git 

```sh
sudo apt-get install git
```

21 Create the application directory 

```sh
cd /var/www
sudo mkdir FlaskApp
cd FlaskApp
```

22 Clone the Catalog App to the server

```sh
sudo git clone https://github.com/chriskelleyGH/catalog_app.git
```

23 Rename the project's name

```sh
 sudo mv ./catalog_app ./FlaskApp
 ```
 
24 Rename application.py to __init__.py

```sh
cd FlaskApp
sudo mv application.py __init__.py
```

25 Update 2 lines of code in __init__.py to refer to the new path of clinet json file

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

26 Edit database_setup.py, __init__.py,data.py, and any other reference to sqlite in a file and change 

```sh
engine = create_engine('sqlite:///catalog.db') to
engine = create_engine('postgresql://catalog:udacity2019@localhost/catalog')
```

27 Install pip 

```sh
sudo apt-get install python-pip
```

28 Use pip to install dependencies (MAKE SURE TO CREATE REQUIREMENTS.TXT FILE)

```sh
$ sudo pip install -r requirements.txt
```

29 Install psycopg2

```sh
sudo apt-get -qqy install postgresql python-psycopg2
```

30 Create database schema

```sh
sudo python database_setup.py
```

31 Configure and Enable a New Virtual Host

Create FlaskApp.conf to edit

```sh
sudo nano /etc/apache2/sites-available/FlaskApp.conf
```

32 Add the following lines of code to the file to configure the virtual host.

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
       
33 Enable the virtual host with the following command

```sh
sudo a2ensite FlaskApp
```

34 Create the .wsgi File under /var/www/FlaskApp:

```sh
cd /var/www/FlaskApp
sudo nano flaskapp.wsgi
```

36 Add the following lines of code to the flaskapp.wsgi file:

```sh
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")
from FlaskApp import app as application
application.secret_key = ’super_secret_key'
```

37 Restart Apache

```sh
sudo service apache2 reload
sudo service apache2 restart
```

38 Configure Google OAuth

* Visit https://console.developers.google.com/apis and obtain Oauth credentials
* Create a new project
* Choose credentials from the menu on the left
* Create an OAuth Client ID
* Configure the consent screen
* When you're presented with a list of application types, choose Web application.  Name project item catalog
* Enter authorized javascript origins
* Enter authorized redirect URI

   Authorized Javascript Origins
   http://18.233.10.243.xip.io

   Authorized Redirect URIS
   http://18.233.10.243.xip.io

* In the Auth Content Screen tab. add xip.io in the Authorized Domains
       
39 Update login.html file with google client ID

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

40 From google developer console, go to project and download JSON.  (Refer To Lesson 6-9 If Necessary) 

41 Rename the file client_secrets.json

42 Replace the client_secrets.json file (from the developer item catalog application) on the LightSail server.   This will be in the same directory as the main python file.

43 Make sure .git directory is not available from the browser.

Put the following text in an .htaccess file at the root of your web server:

```sh
RedirectMatch 404 /\.git
```

### References

The following resources were refernced to complete this project.

https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e

