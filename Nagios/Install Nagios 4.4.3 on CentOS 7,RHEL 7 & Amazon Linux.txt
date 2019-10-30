How To Install Nagios 4.4.3 on CentOS 7,RHEL 7 & Amazon Linux
=================================================

Nagios is the most widely used open source monitoring tools which help us to monitor the services and application that run’s on Windows, Linux, Routers and other network devices.
With the help of Nagios, you can monitor basic services and attributes. We can access the Nagios using web interface coming with the bundle and configuration need to be done on the file level.
Services List
This Tutorial describes how you can monitor private services and attributes of Linux/UNIX servers, such as:

Attributes
=============
•	CPU load
•	Memory usage
•	Disk usage
•	Logged in users
•	Running processes
•	etc.

Private Services
============
•	HTTP
•	FTP
•	SSH
•	SMTP
•	Etc

Provide Security Group
=================
•	HTTP  80
•	SSH   22
•	SMTP  25
•	ICMP IPV4 0-65536 (for ping)
•	TCP Custom: 5666 (for nrpe plugin)

Prerequisites
============
Before installing the Nagios, the system needs to meet the requirements for installing Nagios. So install the Web Server (httpd), PHP, compilers and development libraries.
Install all packages in a single command.
	yum -y install httpd php gcc glibc glibc-common wget perl gd gd-devel unzip zip
Create a nagios user and nagcmd group for allowing the external commands to be executed through the web interface, add the nagios and apache user to be a part of the nagcmd group.
	useradd nagios
	groupadd nagcmd
	usermod -a -G nagcmd nagios
	usermod -a -G nagcmd apache

Install Nagios Server
=================
Download the latest version of Nagios Core using the terminal.
	cd /tmp/
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.3.tar.gz
	tar -zxvf nagios-4.4.3.tar.gz
	cd /tmp/nagios-4.4.3

Compile and Install Nagios.
====================
	./configure --with-nagios-group=nagios --with-command-group=nagcmd
	make all
	make install
	make install-init
	make install-config
	make install-commandmode

Install Nagios Web Interface
=========================
Install the Nagios web configuration using the following command.
	make install-webconf
Run the following command to install a Nagios exfoliation theme
	make install-exfoliation
Create a user account (nagiosadmin) for logging into the Nagios web interface. Remember the password that you assign to this user – you’ll need it later.
	htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
Restart Apache web server to make the new settings take effect.

### CentOS 7 / RHEL 7 ###

	systemctl restart httpd
	systemctl enable httpd

Configure Nagios Server
=======================
Sample configuration files have now been installed in the /usr/local/nagios/etc directory. These sample files should work fine for getting started with Nagios. You’ll need to make just one change before you proceed.
Edit the /usr/local/nagios/etc/objects/contacts.cfg config file with your favorite editor and change the email address associated with the nagiosadmin contact definition to the address you’d like to use for receiving alerts.
vi /usr/local/nagios/etc/objects/contacts.cfg
Change the Email address field to receive the notification.
define contact{
        contact_name                    nagiosadmin             ; Short name of user
        use                             generic-contact         ; Inherit default values from generic-contact template (defined above)
        alias                           Nagios Admin            ; Full name of user

        email                           admin@itzgeek.com       ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
        }

Install Nagios Plugins
===================
Download Nagios Plugins to /tmp directory.
	cd /tmp
	wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
	tar -zxvf nagios-plugins-2.2.1.tar.gz
	cd /tmp/nagios-plugins-2.2.1/
Compile and install the Nagios plugins.
	./configure --with-nagios-user=nagios --with-nagios-group=nagios
	make
	make install

Start Nagios Server
================
Verify the sample Nagios configuration files.
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
Output:
Nagios Core 4.4.3
Copyright (c) 2009-present Nagios Core Development Team and Community Contributors
Copyright (c) 1999-2009 Ethan Galstad
Last Modified: 2019-01-15
License: GPL

Website: https://www.nagios.org
Reading configuration data...
   Read main config file okay...
   Read object config files okay...

Running pre-flight check on configuration data...

Checking objects...
        Checked 8 services.
        Checked 1 hosts.
        Checked 1 host groups.
        Checked 0 service groups.
        Checked 1 contacts.
        Checked 1 contact groups.
        Checked 24 commands.
        Checked 5 time periods.
        Checked 0 host escalations.
        Checked 0 service escalations.
Checking for circular paths...
        Checked 1 hosts
        Checked 0 service dependencies
        Checked 0 host dependencies
        Checked 5 timeperiods
Checking global event handlers...
Checking obsessive compulsive processor commands...
Checking misc settings...

Total Warnings: 0
Total Errors:   0

Things look okay - No serious problems were detected during the pre-flight check

If there are no errors, then start the Nagios service.
	service nagios start
Start Nagios on system startup.
	chkconfig nagios on

Access Nagios Web Interface
=====================
Now access the Nagios web interface using the following URL. You’ll be prompted for the username (nagiosadmin) and password you specified earlier.
http://Nagios-server public-ip-add-re-ss/nagios/

Click on Hosts in the left pane to get a list of systems being monitored by Nagios. We have not added any host to Nagios, So it simply monitors the localhost itself.
Click on Services in the left pane to get the status of any services that are being monitored with Nagios.




