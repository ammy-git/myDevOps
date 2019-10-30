Monitor Remote Linux Systems With Nagios Monitoring Tool
=============================================

NRPE Plugin
===============
Nagios Remote Plugin Executor (abbreviated as NRPE) plugin allows you to monitor applications and services running on remote Linux / Windows hosts. This NRPE Add-on helps Nagios to monitor local resources like CPU, Memory, Disk, Swap, etc. of the remote host.

On Remote Linux System
====================
Switch to the root user.
	sudo su -

Install NRPE Add-on & Nagios Plugins
===============================
NRPE and Nagios plugins are not available in the base repository of CentOS 7. So, you need configure EPEL repository on CentOS 7.
	rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
Use the following command to install NRPE Add-on and Nagios plugins.
### CentOS 7 / RHEL 7 ###
	yum install -y nrpe nagios-plugins-all

Configure NRPE Add-on
=============================
Modify the NRPE configuration file to accept the connection from the Nagios server, Edit the /etc/nagios/nrpe.cfg file.
	vi /etc/nagios/nrpe.cfg
Add the Nagios servers IP address, separated by comma like below.
	allowed_hosts=192.168.1.10   # nagios-server public ip

Configure Nagios Checks
===================
The /etc/nagios/nrpe.cfg file contains the basic commands to check the attributes (CPU, Memory, Disk, etc.architecure) and services (HTTP, FTP, etc.) on remote hosts. Below command lines let you monitor attributes with the help of Nagios plugins.
The path to Nagios plugins may change depends on your operating system architecture (i386 or x86_64).
CentOS 7 / RHEL 7
	command[check_users]=/usr/lib64/nagios/plugins/check_users -w 5 -c 10
	command[check_load]=/usr/lib64/nagios/plugins/check_load -w 15,10,5 -c 30,25,20
	command[check_root]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /dev/mapper/centos-root
	command[check_swap]=/usr/lib64/nagios/plugins/check_swap -w 20% -c 10%
	command[check_total_procs]=/usr/lib64/nagios/plugins/check_procs -w 150 -c 200
In the above command definition -w stands for warning and -c stands for critical.

Test Nagios Checks
====================
For example, execute the below command in another terminal to see the check result.
Tested on Ubuntu 18.04:
	/usr/lib/nagios/plugins/check_procs -w 150 -c 200
Output:
	PROCS WARNING: 190 processes | procs=190;150;200;0;
Nagios plugin will count running processes and will warn you when the process count is more than 150, or it will report you critical when the process count is more than 200, and at the same time, the output will state OK if the count is below 150.
You can adjust the alert level as per your requirements. Change warning to 200 and critical to 250 for testing purpose. Now you can see an OK message.
	/usr/lib/nagios/plugins/check_procs -w 200 -c 250
Output:
	PROCS OK: 189 processes | procs=189;200;250;0;
These command definitions have to be entered on template file on the Nagios server host to enable the monitoring.

Restart the NRPE service.
==================
### CentOS 7 / RHEL 7 ###
	systemctl start nrpe
	systemctl enable nrpe

On Nagios Server
=================
NRPE and Nagios plugins are not available in the base repository of CentOS 7. So, you need configure EPEL repository on CentOS 7.
	rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

Install NRPE plugin
================
Use the following command to install the check_nrpe plugin on your machine.
### CentOS 7 / RHEL 7 ###
	yum -y install nagios-plugins-nrpe
Edit the Nagios configuration file to include all .cfg files inside the /usr/local/nagios/etc/servers directory.
	vi /usr/local/nagios/etc/nagios.cfg
Add or uncomment the following line.
	cfg_dir=/usr/local/nagios/etc/servers
Create a configuration directory.
	mkdir /usr/local/nagios/etc/servers

Configure Nagios Server
=======================
Now it’s time to configure the Nagios server to monitor the remote client machine, and You’ll need to create a command definition in Nagios object configuration file to use the check_nrpe plugin. Open the commands.cfg file.
	vi /usr/local/nagios/etc/objects/commands.cfg
Add the following Nagios command definition to the file.
### CentOS 7 / RHEL 7 ###
# .check_nrpe. command definition

define command{
command_name check_nrpe
command_line /usr/lib64/nagios/plugins/check_nrpe -H $HOSTADDRESS$ -t 30 -c $ARG1$
}

Add a Linux host to Nagios server
=========================
Create a client configuration file /usr/local/nagios/etc/servers/client.itzgeek.local.cfg to define the host and service definitions of remote Linux host.
	vi /usr/local/nagios/etc/servers/client.itzgeek.local.cfg
Copy the below content to the above file.	
You can also use the following template and modify it according to your requirement. The following template is for monitoring logged in users, system load, disk usage (/ – partitions), swap, and total process.

define host{
                           
            use                     linux-server            
            host_name               client.itzgeek.local            
            alias                   client.itzgeek.local            
            address                 192.168.1.20
                                    
}                                   
                                    
define hostgroup{                   
                                    
            hostgroup_name          linux-server            
            alias                   Linux Servers            
            members                 client.itzgeek.local
}                                   
                                    
define service{                     
                                    
            use                     local-service            
            host_name               client.itzgeek.local            
            service_description     SWAP Uasge            
            check_command           check_nrpe!check_swap                          
                                    
}                                   
                                    
define service{                     
                                    
            use                     local-service            
            host_name               client.itzgeek.local            
            service_description     Root / Partition            
            check_command           check_nrpe!check_root                          
                                    
}                                   
                                    
define service{                     
                                    
            use                     local-service            
            host_name               client.itzgeek.local            
            service_description     Current Users            
            check_command           check_nrpe!check_users                         
                                    
}                                   
                                    
define service{                     
                                    
            use                     local-service            
            host_name               client.itzgeek.local            
            service_description     Total Processes            
            check_command           check_nrpe!check_total_procs                   
                                    
}                                   
                                    
define service{                     
                                    
            use                     local-service            
            host_name               client.itzgeek.local            
            service_description     Current Load            
            check_command           check_nrpe!check_load

}

Replacing a value
=================
	sed -i.bak 's/<old-value>/<new-value>/g' <file-path>

	sed -i.bak 's/client.itzgeek.local/ec2-15-206-88-24.ap-south-1.compute.amazonaws.com/g' /usr/local/nagios/etc/servers/client.itzgeek.local.cfg
	sed -i.bak 's/192.168.1.20/15.206.88.24/g' /usr/local/nagios/etc/servers/client.itzgeek.local.cfg

Verify Nagios for any errors.
========================
	/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

Restart the Nagios server.
====================
	systemctl restart nagios

Monitor the remote machine
========================
Go and check the Nagios web interface to view the new services we added just now.
Within a minute, you should start seeing status in services page.

Monitor Apache Service
==================

IN NODE
--------------

1. Install apach httpd
    	yum install httpd -y
    	systemctl enable httpd 
    (still httpd not started)

2. create custom plugin

	vi  /usr/lib64/nagios/plugins/check_apache

#!/bin/bash
service=httpd  #mysql,tree,git
if (( $( ps -ef  | grep -v  grep  |  grep $service  |  grep root |  wc -l ) > 0 ))
then
	echo  "$service is running!!!"; 
	exit 0;
else
	#echo "need to start $service";
	exit 2;
fi

paste following in  ps -ef  | grep -v  grep  |  grep $service  |  grep root |  wc -l    in terminal
check output , it is giving output as zero (0).

also check for the ps -ef  | grep  httpd 
you will get 
root      1872 20535  0 08:11 pts/1    00:00:00 grep --color=auto httpd


3. Assign execute mod to check_apache plugin

	ll  /usr/lib64/nagios/plugins/check_apache
	chmod +x /usr/lib64/nagios/plugins/check_apache
	ll  /usr/lib64/nagios/plugins/check_apache

4. Add check_apache command
	vi /etc/nagios/nrpe.cfg
	:/check_
	command[check_apache]=/usr/lib64/nagios/plugins/check_apache
	save and quit

5. Directly paste following in terminal
	/usr/lib64/nagios/plugins/check_apache
you will get output as "need to start apache"

6. restart nrpe
 	systemctl restart nrpe

IN MASTER
---------------

1. Add service for apache
	 vi /usr/local/nagios/etc/servers/client.itzgeek.local.cfg

define service{

            use                     local-service
            host_name               ec2-15-206-88-24.ap-south-1.compute.amazonaws.com
            service_description     Check Apache 
            check_command           check_nrpe!check_apache

}

2. restart nagios service
 	 systemctl restart nagios

3. Refresh the nagios dashboad browser page
    you will see nagios service is added but in pending state   












