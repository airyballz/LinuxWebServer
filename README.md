# UDACITY'S FULLSTACK WEB DEVELOPER Final Project

## PROJECT DETAILS

Remotley Install A Linux Server To Host A Public Website

## REQUIREMENTS

[Ubuntu][ubuntu]
[Amazon Web Service][aws]
[Amazon LightSail][awsls]
[GIT-Bash][git]

## LINUX PACKAGES USED

Ubuntu
Apache
Flask
Flask-GoogleLogin
SQLAlchemy

## STEPS INVOLVED

### Preparing Your Server

* Download and install GITBash or any other program you can use connect using a SSH terminal session
* Create an AWS (Amazon Web Service) Account
* Create a new instance of Ubuntu using AWS LightSail Server Hosting at Amazon
    - Download a copy of the generated SSH key
    - Navigate to Account section of Amazon LightSail to grab the SSHKeyRename it and remember its location
    - Inside AWS control panel navigate to the network tab for the newly created ninstance
    - Attach a static Ip to your server
    - For security change default firewall entry for SSH port 22 to 2222

### Connecting to Your Server

* We need the following Info to Connect to Your server
    - Rooted User: Ubuntu
    - Public IP: http://55.555.55.55
    - SSH: Port 2222
* Open GIT-bash | Connect To You server
    - GIT-bash in folder where SSH-key was downloaded
    - Enter user, ip, port and Key
        + ssh ubuntu@555.555.55.55 -p 2222 -i SSHkey

### Updating Your Server

* Check For Updates | Installs Then | Reboot
    - apt-get install nano
    - sudo apt-get update
    - sudo apt-get upgrade
    - sudo reboot

### Securing Your Server

* Removes AppArmor And Updates Time
    - echo "Syncronize With Time Server"
    - sudo apt-get -y install ntp ntpdate
* Creates Rules and Enables linux Local Firewall
    - sudo ufw default deny incoming
    - sudo deny ssh
    - sudo ufw default allow outgoing
    - sudo ufw allow 2222/tcp comment 'SSH'
    - sudo ufw allow 80,443/tcp commment 'WWW'
    - sudo ufw allow 8000:8008,8080:8090/tcp comment 'PROXY'
    - sudo ufw allow ntp comment 'TIME'
    - sudo ufw enable
    - sudo service sshd restart

### Add Users To Your Server

* Create New users
    - sudo useradd student --create-home --password "" --shell /bin/bash --uid 5013 --user-group
    - sudo useradd grader --create-home --password "" --shell /bin/bash --uid 5014 --user-group
* Add Student to Sudo
    -  sudo nano /etc/sudoers.d/Users
    - "cat > /tmp/USERS.tmp <<-EOF
    - Added UsersPrivliges rules for student
    - student ALL=(ALL) NOPASSWD:ALL
    - EOF"
    - sudo EDITOR="/tmp/USERS.tmp" visudo -c -q -s -f USERS.tmp
    - sudo chown root:root /tmp/USERS.tmp
    - sudo chmod 0440 /tmp/USERS.tmp
    - sudo mv /tmp/USERS.tmp /etc/sudoers.d/USERS

### Creating SSH Key for Users

* Create Folders For Student
    - su airyman
    - sudo usermod -aG sudo airyman
    - sudo -u airyman mkdir /home/airyman/.ssh
    - sudo -u airyman touch /home/airyman/.ssh/authorized_keys
    - sudo chmod 700 /home/airyman/.ssh
    - sudo chmod 644 /home/airyman/.ssh/authorized_keys
* Create Folders For grader
    - # su grader
    - # sudo usermod -aG sudo grader
    - sudo -u grader mkdir /home/grader/.ssh
    - sudo -u grader touch /home/grader/.ssh/authorized_keys
    - sudo chmod 700 /home/grader/.sshd
    - sudo chmod 644 /home/grader/.ssh/authorized_keys
* Create Keys and Copy to server
    - Open Git locally and run ssh-keygen
    - Enter Key Name and encryption password for student
    - Copy .pub file contents of generated
    - Copy To Server
        + sudo nano //home/student/.ssh/authorized_keys
    - Generate Another Key grader
    - Copy To Server
        + sudo nano //home/grader/.ssh/authorized_keys

### Deploy Project To Your Server

    Install and configure Apache to serve
    sudo apt-get install apache2
    sudo apt-get install libapache2-mod-wsgi python-dev
    sudo a2enmod wsgi
    sudo apt-get install python-pip
    sudo pip install Flask==0.12.2
    sudo pip install SQLAlchemy==1.1.14
    sudo pip install Flask-Login==0.2.11
    sudo pip install Flask-GoogleLogin==0.3.1
    sudo nano /etc/apache2/sites-available/itemcat.conf

        <VirtualHost *:80>
                ServerName ec2-18-217-166-56.us-east-2.compute.amazonaws.com
                ServerAlias 18.217.166.56
                WSGIDaemonProcess itemcat user=www-data group=www-data threads=5
                WSGIScriptAlias / /var/www/itemcat/itemcat.wsgi
                <Directory /var/www/itemcat/itemcat/>
                        WSGIScriptReloading On
                        WSGIProcessGroup itemcat
                        WSGIApplicationGroup %{GLOBAL}
                        Order deny,allow
                        Allow from all
                </Directory>
                Alias /static /var/www/itemcat/itemcat/static
                <Directory /var/www/itemcat/itemcat/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

    sudo a2ensite itemcat
    cd /var/www/itemcat
    sudo nano itemcat.wsgi

        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/itemcat/")
        from itemcat import app as application
        application.secret_key = 'secret key'   

    sudo service apache2 restart
    Install git
     sudo apt-get install git
     git config --global user.name "username"
     git config --global user.email "email@domain.com"

    Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
     git init
     git clone

    Digital Ocean https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts
    Flask Documentation http://flask.pocoo.org/docs/0.12/deploying/
    Various Stack Overflow articles

## RESOURCES


