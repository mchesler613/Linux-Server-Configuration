# Linux Server Configuration Project
## Project Description
This project takes a baseline virtual Linux server deployed from Amazon Web Services (AWS) Lightsail instance and enhances it with security features in order to host a prior Udacity Flask-based database project called **Item Catalog**  on the Internet.

The security features include:
+ disabling remote login as root
+ only allow connections for `SSH` (port 2200), `HTTP` (port 80), and `NTP` (port 123).
+ enforce only key-based SSH authentication
+ updating all system packages to the most recent version
+ a special user **grader** is given SSH access with **sudo** ability

The database server that serves this project is PostgreSQL and the Linux server is configured to host a WSGI (_Web Server Gateway Interface_) version of the **Item Catalog** application.

The following are individual steps taken from start to finish to complete this project.

# Configure Linux Server

## Listing packages  
- View existing installed packages on server via the `/etc/apt/sources.list` command
  
## Update packages
- Type command `sudo apt-get update` to see the packages on server that need an update

## Upgrade packages
- Type command `sudo apt-get upgrade` to update all packages on server

## Auto remove packages that are not required
- Type command `sudo apt-get autoremove` to remove packages that are no longer required

## Install the **finger** package
- Type command `sudo apt-get install finger` to install the **finger** package which provides details about a Linux user

## Create a new user called "grader"
- Type command `sudo adduser grader` to create a new user **grader**
- Type command `finger grader` to view this user

## Give "grader" sudo access
To give the user **grader** access to **sudo**, we need to add **grader** to the /etc/sudoers.d directory.

+ Type command `sudo touch /etc/sudoers.d/grader` to create a blank grader file in the /etc/sudoers.d directory

+ Type command `sudo nano /etc/sudoers.d/grader` and

add this line into the sudoers.d file: `student ALL=(ALL) NOPASSWD:ALL`

## Give "grader" ability to log into server via SSH key pair on default port

### Local Computer

+ generate a secure SSH key-pair via the `ssh-keygen` command on our local computer. Provide the name of the ssh key file as **~/.ssh/UdacityLinux**. Two files will be created in the ~/.ssh directory: **UdacityLinux** and **UdacityLinux.pub**.

### Linux Server

+ as the ubuntu user on the Linux server, login as **grader** with this command `su grader`.

+ create a .ssh directory grader's home directory, /home/grader

+ copy the content of **UdacityLinux.pub** to the file /home/grader/.ssh/authorized_keys via the command `cat > .ssh/authorized_keys` (This content is available in the "Notes to Reviewer" field.)

+ `chmod 700` on the .ssh directory

+ `chmod 644` on the .ssh/authorized_keys file

### Local Computer

+ on our local computer type the following command to SSH into the Linux server as user **grader** `ssh -i ~/.ssh/<private_key> grader@99.79.40.240`


# Force Key-based Authentication; Disable Password Login

To enforce key-based authentication and disable password logins,

+ edit the /etc/ssh/sshd_config file in the Linux server via `vi /etc/ssh/sshd_config` and make sure the line `PasswordAuthentication no` exists. On the Lightsail Linux server, this line is the default.
+ to completely disable **root** login to the server, enter **no** for the `PermitRootLogin no` entry.
  

# Configure Firewall on Linux server

 + Type the following commands:

`    sudo ufw default deny incoming` 
    `sudo ufw default allow outgoing`

 + To allow the ssh on non-default port of 2200, type the following:

`sudo ufw allow 2200/tcp`

+ allow the http on default port 80, type the following:

`sudo ufw allow www`

+ To allow the NTP on default port 123, type the following:

`sudo ufw allow ntp`

+ To enable the firewall configuration, type:

`sudo ufw enable`

+ To check the status of the firewall, type:

`sudo ufw status`

  

The result on my Linux server is as follows:

  

    ubuntu@ip-172-26-13-147:~$ sudo ufw enable
    
    Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
    
    Firewall is active and enabled on system startup
    
    ubuntu@ip-172-26-13-147:~$ sudo ufw status
    
    Status: active
    
    To Action From
    
    -- ------ ----
    
    2200/tcp ALLOW Anywhere
    
    80/tcp ALLOW Anywhere
    
    123 ALLOW Anywhere
    
    2200/tcp (v6) ALLOW Anywhere (v6)
    
    80/tcp (v6) ALLOW Anywhere (v6)
    
    123 (v6) ALLOW Anywhere (v6)

  

# Configure firewall on Amazon Lightsail dashboard

  

+ On the Lightsail Networking tab, add a custom port 2200 as type tcp.

+ Reboot the Lightsail server.

  

# SSH to Linux server

+ Download the private SSH-keygen pair from the Amazon Lightsail dashboard and store it in your local ~/.ssh directory.

+ SSH as **grader** to the Lightsail server at non-default port 2200 by typing:

 `ssh -i ~/.ssh/<private_key> grader@99.79.40.240 -p 2200`

# Install a Web server

## Install Apache

To install the Apache HTTP server, type the following command:

`sudo apt-get install apache2`

To test that a web server is running on port 80, type this command at your local browser or click the link:

[http://99.79.40.240/](http://99.79.40.240/)
  
# Install and configure Database Server

## Install PostgreSQL
We install PostgreSQL using this command: `sudo apt-get install postgresql`

## Create database user `catalog`

Type the command     `sudo -u postgres psql postgres` at the terminal.  The output is as follows:

    psql (9.5.14)
    Type "help" for help.
    
    postgres=# CREATE USER catalog with PASSWORD 'shoresh';
    CREATE ROLE
    postgres=# ALTER USER catalog CREATEDB;
    ALTER ROLE
    postgres=# CREATE DATABASE catalog with OWNER catalog;
    CREATE DATABASE
    postgres=# \c catalog
    You are now connected to database "catalog" as user "postgres".
    catalog=# REVOKE ALL ON SCHEMA public FROM public;
    REVOKE
    catalog=# GRANT ALL ON SCHEMA public TO catalog;
    GRANT
    catalog=# \q

Type the command `psql postgresql://catalog<password>@localhost/catalog` to verify that the database and user are created.

At the **psql** prompt, type `\l` to list databases and users created.  You will see the following:
` 

                                      List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
    -----------+----------+----------+-------------+-------------+-----------------------
     catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
     postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |

`
# Configure local timezone to UTC on linux server

+ type `sudo dpkg-reconfigure tzdata` on terminal and select `None of the above` followed by `UTC`

The output of this command is as follows:

    Current default time zone: 'Etc/UTC'
    Local time is now: Mon Mar 18 04:57:42 UTC 2019.
    Universal Time is now: Mon Mar 18 04:57:42 UTC 2019.
  
# Deploy Item Catalog Project
## Install Git
To install git type this command at the terminal:
`sudo apt-get install git-core`

## Clone Item Catalog project
+ Go to the path `/var/www`
+ `git clone https://github.com/mchesler613/Item-Catalog.git catalog`
+ `chown grader catalog catalog/* catalog/*/*`
+ `chgrp grader catalog catalog/* catalog/*/*`

## Configure Item Catalog project for WSGI
+ To configure Apache to handle requests using the WSGI module, first install mod_wsgi:

`sudo apt-get install libapache2-mod-wsgi`

+ `sudo vi /etc/apache2/sites-available/catalog.conf` 
with this content: 

    `<VirtualHost *:80>
                ServerName 99.79.40.240
                ServerAdmin admin@99.79.40.240
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/static
                <Directory /var/www/catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>`

+ Enable the catalog app with this command: 
`sudo a2ensite catalog`
+ Create the WSGI file: 
`sudo vi /var/www/catalog/catalog.wsgi` 
with the content:

`

    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")
    import app as application
    application.secret_key = 'specialsecretkey'

+ To restart Apache with the new configuration, type `sudo apache2ctl restart`

## Install SQLAlchemy, psycopg2, oauth2client, requests
+ To install **sqlalchemy**, type the command: `sudo -c apt-get install python-sqlalchemy`
+ To install **psycopg2**, type the command: `sudo -c apt-get install python-psycopg2`
+ To install **oauth2client**, type the command: `sudo -c apt-get install python-oauth2client`
+ To install **requests**, type the command: `sudo -c apt-get install python-requests`

## Populate the database with sample data

+ Run the following commands to create and populate the **catalog** database
`
python database_setup.py
python catalogtest.py
`
The sample database contains data for a catalog of clothing categories and items.

+ Verify that the tables exist in the **catalog** database with `psql postgresql://catalog<password>@localhost/catalog`.  At the **psql** terminal type `\dt` and `\d` to list the tables.  

+ Verify that the tables were populated by typing simple queries at the **psql** terminal, such as `select * from category`, `select * from item` and `select * from user`

# Third-party Resources

+ [# How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) 
+ [Flask Documentation](http://flask.pocoo.org/docs/1.0/)
+ [PostgreSQL Documentation](https://www.postgresql.org/docs/9.6/app-psql.html)
+ [Git Hub Help](https://help.github.com/en#dotcom)
+ [Stack Overflow](http://stackoverflow.com)
+ [# Configuring SSHD on the Server](https://serversforhackers.com/c/configuring-sshd-on-the-server)

# Issues
+ At the time of submission of this project in the week of March 18th, 2019, the Google Sign-in Authentication portion of this project is not working.  The Udacity reviewers of this project are already aware of this issue based on this link [Google Sign-in](https://knowledge.udacity.com/questions/28323) submitted by Diego P in the comments.
+ Update: As of March 26, 2019, I checked out this feature and found the Google Sign-In Authentication to be working.  I fixed a couple of errors in the code, used [xip.io](http://xip.io) to turn the Linux server public IP address into a subdomain to increase accessibility on the Web.
+ Update: As of March 26, 2019, I've added a static IP address to the Linux server via Amazon AWS Lightsail and added an A record to my own domain. For instance, I turned the static IP Address into [aws.moriahweb.com] (http://aws.moriahweb.com).
