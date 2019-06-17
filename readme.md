# Linux Server Configuration Project

### About the project
> This is the last project for FSND on Udacity. In this project, a Linux virtual machine needs to be configurated to host my Item Catalog application.

### Project Requirements:
Requirements for this project can be found in [Project Specification](https://review.udacity.com/#!/rubrics/2007/view).

### Access to Application:
*Public IP Address:* 54.184.54.98
*Allpication URL:* http://54.184.54.98
*Accessible SSH Port:* 2200
*Username login:* grader

### Getting Started:
This project uses [Amazon Lightsail](https://aws.amazon.com/lightsail/)

Start a new Ubuntu Linux server instance on Amazon Lightsail.

- Sign up or Log in
- Create an instance by clicking Create Instance button
- Choose Ubuntu (OS only)
- Choose your instance plan (cheapest recommended)
- Keep the default name provided by AWS or rename your instance
- Wait for startup
- Once the instance has started up, follow the instructions provided to SSH into your server

[How to create a server on Amazon Lightsail](https://serverpilot.io/docs/how-to-create-a-server-on-amazon-lightsail)

### Summary of installed software and configurations

1. Download Default Private Key file from Amazon Lightsail / Account / SSH keys section to directory containing your Catalog App
2. In terminal, type:

`$ chmod 600 <name_of_downloaded_file>`

`$ ssh ubuntu@<IP_address> -i <name_of_downloaded_file>`

3. Once logged in update and upgrade installed packages

`$ sudo apt-get update`

`$ sudo apt-get upgrade`

4. Change the SSH port from 22 to 2200

`$ sudo nano /etc/ssh/sshd_config` to open up the configuration file

5. Change the port number from *22* to *2200* in this file

6. Save and exit the file

7. Restart SSH:

`$ sudo service ssh restart`

8. Configure the firewall
**YOU ARE ABOUT TO CHANGE FIREWALL SETTINGS. BE EXTRA CAREFULL WITH THIS PART**

`$ sudo ufw status` firewall status: 

`$ sudo ufw allow 2200/tcp`

`$ sudo ufw allow 80/tcp`

`$ sudo ufw allow 123/udp`

`$ sudo ufw enable`

`$ sudo ufw status` check firewall status

9. Navigate to Amazon Lightsail > [Networking](https://lightsail.aws.amazon.com/ls/webapp/us-west-2/instances/UdacityClassProject/networking) tab, update the firewall configuration by removig default SSH port 22 from the table and add *port 80, 123, 2200*

10. Create a new user account *grader* and give grader sudo access

`$ sudo adduser grader`

`$ sudo nano /etc/sudoers.d/grader`

`$ grader ALL=(ALL) NOPASSWD:ALL` save and exit file

`$ sudo service sshd restart` update file by restart

`$ exit` exit user

11. Create an SSH key pair for *grader* using the `ssh-keygen` tool on your local machine

`$ ssh-keygen` give it a name (e.g. graderKey)

12. at this point two files should bbe created: graderKey and graderKey.pub

13. Open and copy public key

`$ cat graderKey.pub`

14. Navigate back to ubuntu and create .ssh directory for user grader

`$ ssh ubuntu@<IP_address> -i <name_of_downloaded_file>`

`$ cd /home/grader`

`$ sudo mkdir .ssh`

`$ sudo chown grader:grader  .ssh/` change ownership of this direstory to grader

`$ cd .ssh/`

`$ sudo nano authorized_keys` create a file authorized_keys and paste in public key copied in point 13

15. SSH as a *grader* user and install [Apache HTTP Server](https://tutorials.ubuntu.com/tutorial/install-and-configure-apache#0) and Python mod_wsgi

`$ ssh -i graderKey grader@<IP_address>`

`$ sudo apt-get install apache2 libapache2-mod-wsgi python python-pip`

test `$ curl localhost` you should be presented with apache2 default page html file

16. Install PostgreSQL and make sure PostgreSQL does not allow remote connections

`$ sudo apt-get install postgresql`

`$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf`

change line
`local   all   postgres  peer`

Should be
`local   all   postgres  md5`

`$ sudo service postgresql restart` restart PostgreSQL

17. Switch to PostgreSQL defualt user *postgres*:

`$ sudo su - postgres`

`$ psql` open postgresql interactive terminal

`postgres=# CREATE ROLE catalog WITH PASSWORD 'password';` create user *catalog* 

`postgres=# ALTER USER catalog CREATEDB;` allow user to create database 

`postgres=# CREATE DATABASE catalog;` create database

`postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;` 

`postgres=# \c catalog` connect to catalog

`# REVOKE ALL ON SCHEMA public FROM public;`

`# GRANT ALL ON SCHEMA public TO catalog;`

`# \q` exit psql

`postgres=# exit` exit user *postgres*

`$ exit` exit vm

18. Move Item Catalog application directory to the same directory where the server is

19. Open application directory and add file *app.wsgi* with this code. Save it and exit.

`$ cd catalog`

`$ nano app.wsgi`

```
#!/usr/bin/python
import sys
sys.path.insert(0,"/var/www/html/")
from application import app as application
```

20. Install all needed Python modules

`$ ssh -i graderKey grader@<IP_address>`

`$ sudo pip install flask sqlalchemy psycopg2-binary requests oauth2client`

`$ exit`

21. SFTP Item Catalog application

`sftp -i graderKey grader@<IP_address>`

`> put catalog`

`> put -rf catalog`

`> exit`

22. SSH back as a grader and copy application directory to html directory

`$ ssh -i graderKey grader@<IP_address>`

`$ sudo cp catalog/* /var/www/html`

`$ sudo chown -R www-data:www-data /var/www/html/`

`$ sudo chmod -R 774 /var/www/html/`

24. [Configure Apache](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/) to use Python

`$ cd /etc/apache2/sites-enabled/`

`$ sudo nano 000-default.conf`

`WSGIScriptAlias / /var/www/html/app.wsgi` add this script to tell Apache to run a Python app

25. Change database engine in python files in Item Catalog application to:
[reference](https://docs.sqlalchemy.org/en/13/core/engines.html)

`engine = create_engine('postgresql://catalog:password@localhost/catalog')`

26. Run any python code to inialize the database

`$ python <python_files>.py`

27. Restart Apache

`sudo service apache2 restart`

28. Visit IP in browser: http://54.184.54.98


### Troubleshooting
Mostlikely it will not work the first time round.
Keep your cool.

SSH to *grader*

`$ ssh -i graderKey grader@<IP_address>`

Navigate to log file and work through errors 

`sudo cat /var/log/apache2/error.log`


## Good luck!











