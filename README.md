## Linux Udacity's Full Stack Final Project ##

> Configure Linux Webserver

### PROJECT DETAILS ###

> Remotley Install A Linux Server To Host A Public Website      
 http://18.217.166.56/

### REQUIREMENTS ###

> [Ubuntu](https://www.ubuntu.com/)    
[Amazon Web Service](https://aws.amazon.com/)    
[Amazon LightSail](https://lightsail.aws.amazon.com)    
[GIT-Bash](https://git-scm.com/)

### LINUX PACKAGES USED ###

> [Apache2](https://httpd.apache.org/docs/trunk/getting-started.html)    
[Flask 0.12.2](http://flask.pocoo.org/)    
[Flask-Login 0.2.11](https://flask-login.readthedocs.io/en/latest/)    
[Flask-GoogleLogin 0.3.1](https://pythonhosted.org/Flask-GoogleLogin/)    
[SQLAlchemy 1.1.14](https://www.sqlalchemy.org/)    
[WSGI](https://wsgi.readthedocs.io/en/latest/)     


Preparing Your Server
--------------------------------------------------------------------

1. Download and install [GIT-Bash](https://git-scm.com/) or any other program you can use connect using a SSH terminal session
1. Create an [AWS](https://lightsail.aws.amazon.com) (Amazon Web Service) Account
1. Create a new instance of [Ubuntu](https://www.ubuntu.com/) using [AWS LightSail](https://lightsail.aws.amazon.com) Server Hosting at Amazon
    + Download a copy of the generated SSH key
    + Navigate to Account section of [Amazon LightSail](https://lightsail.aws.amazon.com) to grab the SSHKeyRename it and remember its location
    + Inside [AWS](https://lightsail.aws.amazon.com) control panel navigate to the network tab for the newly created ninstance
    + Attach a static Ip to your server
    + For security change default firewall entry for SSH port 22 to 2222

Connecting to Your Server
--------------------------------------------------------------------

1. We need the following Info to Connect to Your server
	+ __Public IP__: http://18.217.166.56/
1. Open [GIT-Bash](https://git-scm.com/) | Connect To You server
	+ [GIT-Bash](https://git-scm.com/) in folder where SSH-key was downloaded
	+ Enter user, ip, port and Key								
		* __sh ubuntu@http://18.217.166.56/ -p 22 -i SSHkey__								

Updating Your Server
--------------------------------------------------------------------

1. Check For Updates | Installs Then | Reboot

	```bash									
	apt-get install nano
	sudo apt-get update
	sudo apt-get upgrade
	sudo reboot
	```									

Securing Your Server
--------------------------------------------------------------------

1. Removes AppArmor And Updates Time

	```bash									
	echo "Removing apparmor and apparmor-utils"
	sudo  dpkg-reconfigure dash
	sudo shell service apparmor stop
	sudo update-rc.d -f apparmor remove
	sudo apt-get remove apparmor apparmor-utils
	echo "Syncronize With Time Server"
	sudo apt-get -y install ntp ntpdate
	```									

1. Creates Rules and Enables linux Local Firewall

	```bash									
	sudo ufw default deny incoming
	sudo deny ssh
	sudo ufw default allow outgoing
	sudo ufw allow 2200 comment 'SSH'
	sudo ufw allow 80/tcp commment 'WWW'
	sudo ufw allow ntp comment 'TIME'
	sudo ufw enable
	```									

1. Disable SSH Passwords | Change SSH Default Port | Disable RootLogin

	```bash									
	sudo nano /etc/ssh/sshd_config
	--------------------------------------------------------------
	Change PasswordAuthentication yes -- PasswordAuthentication
	Change Port 22 -- Port 2200
	Change PermitRootLogin yes -- PermitRootLogin no 
	sudo service sshd restart
	--------------------------------------------------------------
	```									

Add Users To Your Server
--------------------------------------------------------------------

1. Create New users

	```bash									
	sudo useradd student --create-home --password "" --shell /bin/bash --uid 5013 --user-group
	sudo useradd grader --create-home --password "" --shell /bin/bash --uid 5014 --user-group
	```									

1. Add Users to Sudo

	```bash									
	cat > /tmp/USERS.tmp <<-EOF
	#
	# List Of Sudoers Added
	# UsersPrivliges rules for student
	student ALL=(ALL) NOPASSWD:ALL
	# UsersPrivliges rules for grader
	grader ALL=(ALL) NOPASSWD:ALL	
	EOF
	sudo EDITOR="/tmp/USERS.tmp" visudo -c -q -s -f USERS.tmp
	sudo chown root:root /tmp/USERS.tmp
	sudo chmod 0440 /tmp/USERS.tmp
	sudo mv /tmp/USERS.tmp /etc/sudoers.d/USERS
	```									

Creating SSH Key for Users
--------------------------------------------------------------------

1. Create Folders For Student

	```bash									
	sudo usermod -aG sudo student
	sudo -u airyman mkdir /home/student/.ssh
	sudo -u airyman touch /home/student/.ssh/authorized_keys
	sudo chmod 700 /home/student/.ssh
	sudo chmod 644 /home/student/.ssh/authorized_keys
	```									

1. Create Folders For grader

	```bash									
	sudo -u grader mkdir /home/grader/.ssh
	sudo -u grader touch /home/grader/.ssh/authorized_keys
	sudo chmod 700 /home/grader/.ssh
	sudo chmod 644 /home/grader/.ssh/authorized_keys
	```									

1. Create Keys and Copy to server
	+ Open [GIT-Bash](https://git-scm.com/) locally and run ssh-keygen
	+ Enter Key Name and encryption password for student
	+ Copy .pub file contents of generated
	+ Copy To Server
		```bash									
		sudo nano //home/student/.ssh/authorized_keys
		```									
	+ Generate Another Key grader
	+ Copy To Server
		```bash									
		sudo nano //home/grader/.ssh/authorized_keys
		```									

Deploy Project
--------------------------------------------------------------------

1. Install Required Plugins

	```bash									
	sudo apt-get install apache2
	sudo apt-get install libapache2-mod-wsgi python-dev
	sudo a2enmod wsgi
	sudo apt-get install python-pip
	sudo pip install Flask==0.12.2
	sudo pip install SQLAlchemy==1.1.14
	sudo pip install Flask-Login==0.2.11
	sudo pip install Flask-GoogleLogin==0.3.1
	```									

1. Configure [Apache](https://httpd.apache.org/docs/trunk/getting-started.html)

	```bash									
	sudo nano /etc/apache2/sites-available/itemcat.conf
	-------------------------------------------------------------
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
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	-------------------------------------------------------------
	sudo a2ensite itemcat
	```										
	
1. Configure [WSGI](https://wsgi.readthedocs.io/en/latest/) File

	```bash									
	sudo nano itemcat.wsgi
	-------------------------------------------------------------
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/itemcat/")
	from itemcat import app as application
	application.secret_key = 'secret key'
	-------------------------------------------------------------
	sudo service apache2 restart
	```									

1. Install [GIT-Bash](https://git-scm.com/)

	```bash									
	sudo apt-get install git
	git config --global user.name "username"
	git config --global user.email "email@domain.com"
	```										

1. Clone Item Catalog Repository from Previous Project

	```bash									
	git init
	git clone
	```									

## RESOURCES
--------------------------------------------------------------------

[How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

[How To Setup a Firewall with UFW on an Ubuntu and Debian Cloud Server](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)

[New Ubuntu 14.04 Server Checklist
](https://www.digitalocean.com/community/tutorial_series/new-ubuntu-14-04-server-checklist)

[How To Serve Django Applications with Apache and mod_wsgi on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-apache-and-mod_wsgi-on-ubuntu-14-04)

[How to Install Laravel with an Nginx Web Server on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-laravel-with-an-nginx-web-server-on-ubuntu-14-04)

[The Perfect Server - Ubuntu 16.04 (Nginx, MySQL, PHP, Postfix, BIND, Dovecot, Pure-FTPD and ISPConfig 3.1)](https://www.howtoforge.com/tutorial/perfect-server-ubuntu-with-nginx-and-ispconfig-3/)