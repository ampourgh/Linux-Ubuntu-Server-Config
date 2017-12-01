#### Author: ampourgh

# Project 6 — flaskapp-server Ubuntu server
### located: Ohio, Zone A (us-east-2a)



## Table of Contents
#### I. Lightsail server & Git Information
I.I. IP address and SSH port 

I.II. URL to Hosted Web Application

I.III. Summary of Software Installed and Configuration Changes Made

I. IV. Third-party Resources

#### II. Steps Taken to Deploy Webpage



## I - Lightsail server & Git Information

### I.I. IP address and SSH port 

#### IP Address

Public IP: 52.14.27.203

#### To SSH as ubuntu to Amazon Lightsail
```
ssh ubuntu@52.14.27.203 -p 2200 -i LightsailPrivateKey.pem
```

#### To SSH as the grader using a keygen
```
ssh grader@52.14.27.203 -p 2200 -i ~/.ssh/graderKeygen
```
There is no pass phrase for the keygen, however the server password for the grader is grader.

### I.II. URL to Hosted Web Application

URL to webpage: http://ec2-52-14-27-203.us-east-2.compute.amazonaws.com
URL to AWS account page: https://lightsail.aws.amazon.com/ls/webapp/home/instances

### I.III. Summary of Software Installed and Configuration Changes Made

#### Ubuntu Firewall ports

Sudo UFW (Ubuntu Firewall) is currently active, and allows for SSH port 2200, HTTP/www port 80 and NTP port 123. PostgreSQL is also active on port 5432 specifically for the use of this server instance. To check the status and active ports on the server, use the command 'sudo ufw status'.

#### Apache2 

Apache2 has been installed to serve the website, which includes the directory for what websites could be and are being served.

#### Changing Webpages showing up on the IP address

After getting Apache2 installed, /etc/apache2/ has the folders related to putting the webpages on port 80. The folders include sites-available and sites enabled. Currently, there are two files available that boots up different pages. 000-default.conf connects the default ubuntu page, and FlaskApp.conf boots up the items catalog page.

In order to connect a site to a port, use the following command:
```
sudo a2ensite <insert virtual host file name>
```
To disconnect the site:
```
sudo a2dissite <insert virtual host file name>
```

After connecting or disconnecting, Apache2 will need to be reloaded in order to update the service:
```
sudo service apache2 reload
```

#### Flask App's wsgi File

From the previously discussed FlaskApp.conf files, the WSGIScriptAlias is connected to 'flaskapp.wsgi'. This file gives the basic configuration to run the flask app. Below is the code for the modifiable sections of the wsgi that connects it between one app to another, along with the place for were Lightsail's secret key is inserted. 

```python
#!/usr/bin/python
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

// from /var/www/FlaskApp/FlaskApp/__init__.py
// import app = Flask(__name__)
from FlaskApp import app as application
// requires quotation marks for multlined private key
application.secret_key = """<insert LightsailPrivateKey here>"""
```

For Python 2 or lower, According to Flask's doc: "This sets up the load paths according to the settings of the virtual environment."
```python
activate_this = '/var/www/FlaskApp/FlaskApp/venv/bin/activate_this.py'
exec(compile(open(activate_this, "rb").read(), activate_this, 'exec'), dict(__file__=activate_this))
```

#### Python3-flask, pip3

Python3's Flask and pip3 are installed on Ubuntu, along with the Virtual Environment, but the project is currently running without the activating venv. The files below include the libraries, tools and framework used to serve the catalog.  

```
Flask (0.12.2)
httplib2 (0.10.3)
Jinja2 (2.10)
oauth2client (4.1.2)
pip (9.0.1)
psycopg2 (2.7.3.2)
requests (2.18.4)
SQLAlchemy (1.1.15)
urllib3 (1.22)
Werkzeug (0.12.2)
```

### I.IV. Third-party Resources

None.



## Steps Taken to deploy webpage:

### Registering for an account at Lightsail
* Head to lightsail.aws.amazon.com
* Click to create an instance, choose the choice for a Linux platform, with an OS only of Ubuntu.
* Create and download the secret key for logging onto the server, transferring content, and for connecting websites. The name of the secret key in this project is called 'LightsailSecretKey.pem'.
* Choose a payment plan and give your instance a name.

### To Change the SSH Port from a Linux Server
* Connect to instances' browser terminal
* Use command 'sudo su -' to change to the root user.
* vi /etc/ssh/sshd_config to view the SSH configuration, and click 'i' to be able to modify the page's information.
* locate the line 'Port 22' and change the port to 2200.
* Exit the browser terminal.
* Click to manage the instance and delete the SSH port, add custom ports 2200 and 123.

### Connecting remotely and copying files over to Apache2:
* First, I added the flaskapp.wsgi file in the folder where my app is, for convenience. Information on the contents of the .wsgi file can be lower later in this readme.
* Change directories to where your SecretKey.pem file is located.
* Use the command below to transfer the file over to your account's folder. The breakdown of this command includes the scp to copy the file, -i PrivateKey.pem to access as the user, -P 2200 to specify the port, and /c/path/to/folder/ to copy the desired folder. The last section includes your username@public.port.number:/folder/you/want/to/copy/the/files/to.
```
scp -i LightsailPrivateKey.pem -P 2200 -r /c/path/to/folder/ ubuntu@52.14.27.203:/home/ubuntu
```
* The following command is for logging into the page:
```
ssh ubuntu@52.14.27.203 -p 2200 -i LightsailPrivateKey.pem
```
* To install Apache2, the server needs to be up to date so that the www folder is created for storing the webpage's contents. After the Linux server is updated, install Apache2.
```
apt-get update
sudo apt-get install apache2
```
* Created a directory will house the FlaskApp in /www, move the webpage's folder there, followed by moving the .wsgi outside the webpage folder.
```
mkdir /var/www/FlaskApp
mv /home/ubuntu/FlaskApp /var/www/FlaskApp
mv ./flaskapp.wsig ../
```

### Installing wsgi and the virtual environment, and activating the virtual environment.

* Now use the command to install mod-wsgi-p3, the '-p3' extension added for Python 3+.  
```
sudo apt-get install libapache2-mod-wsgi-py3
```

* Go to Apache2's folder for seeing what sites are enabled for viewing on the ip. 
```
cd /etc/apache2/sites-enabled/
```

* If you see 000-default.conf enabled, or anything outside of FlaskApp.conf, use the following commands to switch what site port 80 is connected to.
```
sudo a2dissite 000-default.conf
sudo a2ensite FlaskApp.conf
service apache2 reload
sudo service apache2 restart
```

* Next, use the built in install command to install pip.
```
sudo apt-get install python-pip 
```

* Switch back to the Flask app folder to install and activate a virtual environment.
```
cd /var/www/FlaskApp/FlaskApp
apt-get install python3-venv
source venv/bin/activate 
```

* Install the following libraries/toolsets so that __init__.py can run with the imports.
```
pip install Flask SQLAlchemy psycopg2 requests oauth2client httplib2
```

* At the end pip will mention that the version it's running is not the latest version. Update 
```
>You are using pip version 8.1.1, however version 9.0.1 is available.
pip install --upgrade pip
```

* See if the webpage is served within the virtual environment.
```
python __init__.py
*Deactivate the virtual environment.
deactivate
```

### Google Oauth2 API Credentials
With the transition from localhost to AWS, the oauth2 credentials need to both be updated, this includes withinn client_secrets and within [Google APIs' dashboard](https://console.developers.google.com/apis/credentials?project=archive-173500).

The changes within client_secrets.json include the following:

```json
    "redirect_uris":["http://ec2-52-14-27-203.us-east-2.compute.amazonaws.com/callback","https://ec2-52-14-27-203.us-east-2.compute.amazonaws.com/callback"],
    "javascript_origins":["http://ec2-52-14-27-203.us-east-2.compute.amazonaws.com"]
```

### Postgresql setup

The following steps were taken to set up postgresql database:

```
# first, install the database onto Ubuntu
sudo apt-get install postgresql postgresql-contrib

# create the catalog user within the database, giving them limited administrative capabilities
sudo -u postgres createuser --interactivn
>Enter name of role to add: sammy
>Shall the new role be a superuser? (y/n) n

# now create the user for Ubuntu
sudo adduser catalog

# Create the database as well
createdb catalog

# access postgresql with the command psql, but here we want to specifiy that it's with the catalog user
sudo -u catalog psql

# change the password for catalog for the engine requirement
\password
>Enter new password: catalog
>Enter it again: catalog

#quit psql
\q
```

Now with both database_setup.py and __init__.py, you will specify the the information the engine will use, including the user, password and the database name. 

```
# login to postgresql by user catalog : pass catalog, changed form sqlite from earlier project
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
Base.metadata.create_all(engine)
```

Once done, use the command python database_setup.py to apply the tables into the database. You can now use psql command from before to log onto the database and use either command \d or \dt to display the contents on the database.


### Creating Grader user and SSH keygen

To create a user, use the following command and give the user a password and other information.

```
sudo adduser grader
```

Once in /home/grader, create an .ssh directory followed by a autorized key file.

```
mkdir .ssh
source .ssh/authorized_keys
ch
```

Give the user the authorization to login. 
```
chmod 700 /home/grader/.ssh
chmod 600 /home/grader/.ssh/authorized_keys
chown -R grader:grader .ssh
```


Outside of the server, change directory to /c/users/YourDesktopName/ and type 'ssh key-gen', follow by the directory the key-gen will be placed. The SSH passphrase for grader has been kept empty.

```
ssh-keygen
> Enter file in which to save the key (/c/Users/Jamshid/.ssh/id_rsa): /c/Usersshid/.ssh/graderKeygen
> Enter passphrase (empty for no passphrase):
> Enter same passphrase again:
> Your identification has been saved in /c/Users/Jamshid/.ssh/graderKeygen.
> Your public key has been saved in /c/Users/Jamshid/.ssh/graderKeygen.pub.
```

As the ubuntu/root user, switch over as grader, and copy the pub file contents into /home/grader/.ssh/authorized_keys. 

```
su - grader
pwd
>/home/grader
vi .ssh/authorized_keys
```
Now login as the grader from /c/users/YourDesktopName/:

```
ssh grader@52.14.27.203 -p 2200 -i ~/.ssh/graderKeygen
```

### Vim cookbook:
* View file in Vim: vi insert-filename
* Modify text: 'i' 
* Save and quit:':' followed by 'wq' 
* Quit without editing: ':q!'
* Scroll Faster: press and hold either shift or use the number while pressing the up/down arrow key pad.
Additional commands for Vim can be found [here](https://vim.rtorr.com).

## Acknowledgements 
1. [SQLAlchemy documentation using various endgines.](http://docs.sqlalchemy.org/en/latest/core/engines.html)
2. [How to deploy a flask app on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
3. [How to install postgresql on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
4. [UFW Essential common firewall rules](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands)
5. [Configuring a SSH key](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-freebsd-server)
6. [postgresql create role documentation](https://www.postgresql.org/docs/8.2/static/sql-createrole.html)
