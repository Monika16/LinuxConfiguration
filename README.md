# LinuxConfiguration
Deploying ItemCatalog application on a secure web server. Used Amazon Lightsail to deploy ItemCatalog. 

## Accessing the Server
IP Address: `54.245.163.212` SSH Port: 2200

## Url for the application
http://54.245.163.212.xip.io/

## Changed Configuration
Created a Lightsail Instance by logging into AWS Amazon account.
Connect to the Lightsail Instance.
Update the software:
```
sudo apt-get update
sudo apt-get upgrade

```

Enable Firewall:
Change the ssh port from 22(default) to 2200.
```
sudo ufw allow ssh
sudo ufw allow 2200\tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable

```
## Grader User
```
Created user grader: `sudo adduser grader`.
Gave `sudo` privilages: Added file `sudo touch grader` in \etc\sudoers.d. Made rule `grader ALL=(ALL) NOPASSWD:ALL` in grader file.
Grader user is assigned a unique key pair to access the ssh.
Make .ssh directory in `sudo mkdir /home/grader/.ssh`.
Create a file authorized_keys in grader .ssh directory `sudo touch authorized_keys`.
Create ssh key pair using `ssh-keygen`.
Copy the .pub key generated using ssh-keygen to authorized_keys file in grader/.ssh.
Now login as grader `ssh -i ~/.ssh/grader grader@54.245.163.212 -p 2200`
```
## Install PostgreSql
Configure PostgreSql `sudo apt-get install postgresql`
Created Database instance in Amazon RDS. 
Database access is done by `'postgresql://catalog1:password@mydata.cef6edmtz6sl.us-west-2.rds.amazonaws.com:5432/itemCatalog'`

## Software Installed
Install Apache Server
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi

```
restart apache 
`sudo service apache2 restart`

Install Flask
```
sudo apt-get install python-pip
sudo pip install Flask
sudo pip install Flask-SQLAlchemy
sudo pip install oauth2client
sudo pip install requests
sudo pip install psycopg2-binaries

```
Install git
`sudo apt-get install git`

In \var\www\itemCatalog clone the project `https://github.com/Monika16/Catalog-App.git`

## Configure Apache
`sudo nano /etc/apache2/sites-available/itemCatalog.conf`
Paste the following code:
```
<VirtualHost *:80>
        ServerName      54.245.163.212
        ServerAdmin     monikathokala16@gmail.com

        WSGIScriptAlias / /var/www/itemCatalog/Catalog-App/itemCatalog.wsgi

        <Directory /var/www/itemCatalog/Catalog-App>
                Order allow,deny
                Allow from all
        </Directory>
        <Directory /var/www/itemCatalog/Catalog-App/static>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
Configuring itemCatalog.wsgi file
create itemCatalog.wsgi in /var/www/itemCatalog/Catalog-App/ directory
```
#!/user/bin/python
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/itemCatalog/Catalog-App')

from project import app as application
application.secret_key='super_secret_key'

```
`sudo a2ensite itemCatalog.conf`

## OAuth authentication
In **https://console.developers.google.com** and to our project
Add `	http://54.245.163.212.xip.io` to Authorised JavaScript origins 
Add ` http://54.245.163.212.xip.io/login` and `http://54.245.163.212.xip.io/gconnect` to Authorised redirect URIs
Save it and download the client_secret.json. Update the same in the project at /var/www/itemCatalog/Catalog-App directory.

Update the client_secret.json in project.py file 

```
CLIENT_ID = json.loads(
    open('/var/www/itemCatalog/Catalog-App/client_secrets.json', 'r').read())['web']['client_id']
oauth_flow = flow_from_clientsecrets('/var/www/itemCatalog/Catalog-App/client_secrets.json', scope='')

```
## Third-party Resources 

1. Creating and Working on Amazon Lightsail Instance: https://docs.aws.amazon.com/cloud9/latest/user-guide/lightsail-instances.html. 
2. Creating and Working on Amazon RDS for database: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html.
3. Creating user and giving sudo privilages and setting Firewall: https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175461/lessons/4331066009/concepts/48010894990923.
4. Deploying the project on Amazon Lightsail: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps.
5. Editing Readme.md: https://help.github.com/articles/basic-writing-and-formatting-syntax/.
