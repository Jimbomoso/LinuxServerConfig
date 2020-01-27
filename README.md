
# LinuxServerConfig 

    App IP address: 34.226.107.237

    Port: 2200

URL: http://34.226.107.237

Udacity Full-Stack Project

## Background 

This was a required project for my Udacity Full-Stack Program. The prior project was creating a full-stack CRUD app using vagrant, virtualbox, python, flask, and postgresql. This project builds on that by taking that and configuring a linux server on LightSail to host the app.

## Steps

### 1) Create a LightSail instance

* The first thing you will need to do is a create an [aws account](http://aws.amazon.com). If you already have an Amazon account you can use those credentials to log in if you'd like or you can create an account from scratch.

* Once logged in to aws locate the LightSail product. At the time of this writing it can be found by clicking the compute button under the explore our products section or by using  the search magnifying glass icon and searching for LightSail.

* On the main LightSail page click create new instance. This will bring you to another page where you will need to select from a bunch of options. For this project I defered to Udacity and selected:

#### Platform

    Linux

#### Blueprint

    OS Only

* Ubuntu 16.04 LTS

* Then select a plan (I choose the cheapest which says the first month's free).

* Next you can choose whether to keep the suggested instance name or to create your own.

* Next click create instance.

* You're instance will initially be grayed out and say pending. After not that long it will ungray and say running.

### 2) Download aws ssh key

This is not intutive.
  
* At the top of the same screen click on the account dropdown and select account.

* There should three options on the screen; profile, SSH keys and Advanced. Select SSH keys.

* Now there should be a download option. Click it and choose whether to keep or rename the file and where to save the file. (I kept the default and downloaded to my download file)

* Now open up your local terminal and change its permissions of the file by typing:

        chmod 600 theNameOfTheFileYouDownloaded

chmod 600 changes the file's permissions to owner can read and write

example:

    chmod 600 LightSailDefaultPrivateKey-us-east-1.pem

### 3) Connecting to your instance

I recommend connecting to the instance using your terminal and avoiding LightSail's option to connect in LightSail, which opens up a SSH connection in your browser, altogether.

* So in your terminal type:

        ssh ubuntu@yourInstancesPublicIPAddress -p 22 -i ~/pathToYourPrivateKey

    example:

        ssh ubuntu@5.190.145.76 -p 22 -i ~/Downloads/LightSailDefaultPrivateKey-us.east-1.pem

If you get any sort of error in the terminal check that you have the correct filename and path. If the error persists you can start over by deleting the instance.  To do so:

* go to your instance on the LightSail page
* click the elypsis button
* select delete and confirm.
  
At this point deleting the instance is no big deal.

### 4) Install updates

Once connected to your instance in the terminal it is a good idea to install updates.

    sudo apt-get update

    sudo apt-get upgrade

You will get asked if it is OK to install stuff. You have to select yes to do so. Once complete another screen asked me something about whether to keep the local version or some other options. I choose to keep the local version.

* Automatically install updates

        sudo apt-get install unattended-upgrades 
        sudo nano /etc/apt/apt.conf.d/50unattended-upgrades

* Add these lines to the now open file's existing two lines

    APT::Periodic::Download-Upgradable-Packages "1";
    APT::Periodic::AutocleanInterval "7";

* Enable and restart apache 
  
        sudo dpkg-reconfigure --priority=low unattended-upgrades
        sudo service apache2 restart

### 5) Create/setup user grader

* Create user

        sudo adduser grader

* Create grader file in sudo

        sudo touch /etc/sudoers.d/grader

You can confirm by making sure grader is listed by

    sudo ls /etc/doers.d

* Open that file to add sudo access

        sudo nano /etc/sudoers.d/grader

* The file should be blank. Enter: 

    grader ALL=(ALL) NOPASSWD:ALL

* Exit the file with control+X, select yes to save and confirm the filename (do not change it).

### 6) Create SSH key pair for grader

I am using a Mac. I'm not sure if the steps are different for other OSes. In this step you will use your computer's built in SSH key generator to generate a public and private key for the user, grader, that we created earlier.

* In the terminal type:

        ssh-keygen

* I named the keys:

        linuxServerConfig

The keygen created two files:

    linuxServerConfig (private key)
    linuxServerConfig.pub (public key)

* Make sure the private key is in your local machine's .ssh directory.  On mac this is /Users/mac/.ssh and they keys went here by default. I moved the public key to my downloads folder.
  
* In the connected terminal switch to the grader user

        su - grader

* Make a directory called .ssh and a file called authorized_keys
  
        mkdir .ssh
        touch .ssh/authorized_keys

* On your computer open the public key file. To do this I had to: find the file in my downloads folder, right click on it, choose Open With and choose VisualStudioCode to be able to read the file.

* Once open, copy the contents of the public key file.
  
* Open the authorized_keys file in the terminal by typing:

        sudo nano .ssh/authorized_keys

* Paste the copied public key into the open file in the terminal then press control+X, yes to save and return to confirm the filename.
  
* Set the permission levels for the .ssh folder (read/right/execute for user) and authorized_keys file (read/write user, read group, read other)

        chmod 700 .ssh
        chmod 644 .ssh/authorized_keys

### 7) Update the SSH port

* Back in LightSail click the elypsis on your instance and select manage 
  
* From there scroll down to firewall and click add another
  
* Choose Custom/TCP/and enter 2200

* Save

### 8) Configure uncomplicated firewall

* Open server config file back in the terminal and change port to 2200, then restart

        sudo nano /etc/ssh/sshd_config
        sudo service ssh restart

* Configure firewall to allow only incoming connections to HTTP/NTP/2200 and enable the fireware.

        sudo ufw allow 2200/tcp
        sudo ufw allow www
        sudo ufw allow 123/udp
        sudo ufw deny 22
        sudo ufw default deny incoming
        sudo ufw default allow outgoing
        sudo ufw enable

* To check the status of firewall:

        sudo ufw status

### 9) SSH into grader user

Now we have everything set up that we need to to be able to login as user grader whenever we want.  To login via the terminal:

    ssh grader@thePublicIPAddressOfYourInstance -p 2200 -i ~/thePathOfYourConfigFile

example:

    ssh grader@34.226.107.237 -p 2200 -i ~/.ssh/linuxServerConfig

### 10) Configure timezone

        sudo cat /etc/timezone

* if not already UTC, set to UTC

        sudo dpkg-reconfigure tzdata

### 11) Install/configure Apache and wsgi package

     sudo apt-get install libapache2-mod-wsgi
     sudo apt-get install apache2

### 12) Install/configure PostsgreSQL

    sudo apt-get install postgresql
    sudo cat /etc/postgresql/9.5/main/pg_hba.conf
    sudo service apache2 restart

* Create database

        sudo su - postgres
        CREATE USER catalog WITH PASSWORD 'catalog';
        \du
        CREATE DATABASE catalog;
        \l

### 13) Deploy app from git and create setup on server

        cd /var/www
        sudo mkdir FLaskApp
        cd FlaskApp
        sudo git clone (URI for your repo)
        sudo apt install tree
        tree

### 14) Set up virtual environment and install dependencies 

        sudo apt-get install python3-pip
        sudo apt-get install python-virtualenv
        sudo virtualenv -p python3 venv
        pip install httplib2
        pip install python3 requests
        pip install python3 oauth2client
        pip install python3 sqlalchemy
        pip install flask
        sudo apt-get install libpq-dev
        pip install python3 psycopg2

### 15) Enable FlaskApp

* Open configuration file

        sudo nano /etc/apache2/sites-available/FlaskApp.conf 

* Change contents to: 

        sudo nano /etc/apache2/sites-available/FlaskApp.conf

        <VirtualHost *:80>
            ServerName 52.26.30.44
            ServerAdmin sarithakamath24@gmail.com
            WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
            <Directory /var/www/FlaskApp/ItemCatalog/>
                Order allow,deny
                Allow from all
            </Directory>
            Alias /static /var/www/FlaskApp/ItemCatalog/static
            <Directory /var/www/FlaskApp/ItemCatalog/static/>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
            </VirtualHost>

* Enable

        sudo a2ensite FlaskApp

* Create the .wsgi (said whiskey) file

        cd /var/www/FlaskApp/itemCatalog
        sudo nano flaskapp.wsgi

* Add the following to the contents:

        #!/usr/bin/python3
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/itemCatalog/")

        from application import app as application
        application.secret_key = 'super_secret_key'

* Restart apache2

        sudo service apache2 restart

* Change apache2 site

        sudo a2dissite 000-default
        service apache2 restart

### Resources

    https://help.ubuntu.com/
    https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
    https://www.postgresql.org/docs/current/static/sql-createuser.html
    http://flask.pocoo.org/docs/0.12/
    Udacity mentor Michael A.
