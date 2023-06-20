# Linux network services

## Project context 
The local library in your little town has no funding for Windows licenses so the director is considering Linux. Some users are sceptical and ask for a demo. The local IT company where you work is taking up the project and you are in charge of setting up a server and a workstation.
To demonstrate this setup, you will use virtual machines and an internal virtual network (your DHCP must not interfere with the LAN).


You may propose any additional functionality you consider interesting.

## Must Have

Set up the following Linux infrastructure:

1. One server (no GUI) running the following services:
    - DHCP (one scope serving the local internal network)  isc-dhcp-server
    - DNS (resolve internal resources, a redirector is used for external resources) bind
    - HTTP+ mariadb (internal website running GLPI)
    - **Required**
        1. Weekly backup the configuration files for each service into one single compressed archive
        2. The server is remotely manageable (SSH)
    - **Optional**
        1. Backups are placed on a partition located on  separate disk, this partition must be mounted for the backup, then unmounted

2. One workstation running a desktop environment and the following apps:
    - LibreOffice
    - Gimp
    - Mullvad browser
    - **Required** 
        1. This workstation uses automatic addressing
        2. The /home folder is located on a separate partition, same disk 
    - **Optional**
        1. Propose and implement a solution to remotely help a user



## *On my Virtual box I installed 2 machines, Ubuntu server and client Kali,*
## *I set up settings on Bridget Adapter on both machines.*

# *1. Giving IP address to my Ubuntu Server:*
	command: 
	  >	sudo nano 00-installer-config.yaml
	add:
	network:
	  ethernets:
	    enp0s3:
			addresses: [192.168.2.4/24]
			gateway4: 192.168.2.1
			nameservers:
				addresses: [8.8.8.8, 8.8.4.4]
		  dhcp4: no
	  version: 2
      renderer: networkd
# *2. Install Firewall, Harden ssh, Client config, Server config, Test connection:*
	I used documentation: Server_hardening

# *3. Installation and configuration DHCP server on Ubuntu machine:*
	
	command:
	  >	-ip -a: 
		 enp0s3 ip= 192.168.2.4/24
	command: 
	  >	sudo apt install -y isc-dhcp-server
	command: 
	  >	sudo nano /etc/default/isc-dhcp-server
	add:
		INTERFACESv4="enp0s3"
	command:
	  >	sudo nano /etc/dhcp/dhcpd.conf
	add:
		option domain-name "example.org";
		option domain-name-servers server.example.org;
		default-lease-time 600;
		max-lease-time 7200;
		ddns-update-style none;
	
		subnet 192.168.2.0 netmask 255.255.255.0 {
		range 192.268.2.20 192.168.2.254;
		option domain-name-servers server.example.org;
		option domain-name "example.org";
		option subnet-mask 255.255.255.0;
		option routers 192.168.2.1;
		option broadcast-address 192.168.2.255;
		default-lease-time 600;
		max-lease-time 7200;
	command:
	  >	sudo systemctl restart isc-dhcp-server
	  >	sudo systemctl status isc-dhcp-server
	Checking port for dhcp: 
	command: 
	  >	sudo ss -ulpn | grep dhcp
		output:
			dhcp-port 67
		 
	Adding port to my firewall:
	command: 
	  >	sudo ufw allow 67/udp
	command: 
	  >	sudo ufw status
	
# *4. Installation and configuration DNS server on my Ubuntu machine:*
	command: 
	  > sudo apt update
	  > sudo apt install bind9 bind9-utils bind9-dnsutils -y
		 // Check if named service enabled
      > sudo systemctl is-enabled named
		 // Check named service status
	  >	sudo systemctl status named
	  > sudo nano /etc/default/named:
		add:
			OPTIONS="-4 -u bind"
	  >	sudo nano /etc/bind/named.conf.options:
		add:
			// listen port and address
			listen-on port 53 { localhost; 192.168.2.4; }
			
			// for public DNS server - allow from any
			allow-query { any; };
			
			// define the forwarder for DNS queries
			forwarders { 1.1.1.1; }; 
			 
			// enable recursion that provides recursive query
			recursion yes;
			
	Verify the BIND configuration:
		
		command: 
		  >	sudo named-checkconf
		
		If there’s no output, the BIND configurations
		are correct without any error.
			
	Setting Up DNS Zones:
	
	command: 
	  >	sudo nano /etc/bind/named.conf.local :
		add:
		zone "example.org" {
			type master;
			file "/etc/bind/zones/forward.example.org";
		};
		
		zone "4.2.168.192.in-addr.arpa" {
			type master;
			file "/etc/bind/zones/reverse.example.org";
		};
		
	Save the changes and close the file.
	
	Create a new directory () for string DNS zones configurations.
	/etc/bind/zones
	
	command:
	  >	sudo -mkdir -p /etc/bind/zones/   
	
	Copy the default forward and reverse zones 
	configuration to the directory:
	commands:
			// Copy default forward zone
			  >	sudo cp /etc/bind/db.local /etc/bind/zones/forward.atadomain.io

			// Copy default reverse zone
			  >	sudo cp /etc/bind/db.127 /etc/bind/zones/reverse.atadomain.io

			// List contents of the /etc/bind/zones/ directory
			  >	ls /etc/bind/zones/
				
	Edit the forward zone configuration.  This configuration will translate
	the domain name to the correct IP address of the server:
	
	command: 
	  >	sudo nano /etc/bind/zones/forward.example.org:
	add:
				;
				; BIND data file for the local loopback interface
				;
				$TTL    604800
				@   IN      SOA     example.org. root.example.org. (
											2         ; Serial
										604800         ; Refresh
										86400         ; Retry
										2419200         ; Expire
										604800 )       ; Negative Cache TTL

				; Define the default name server to server.example.org
				@   IN      NS      server.example.org.

				; Resolve ns1 to server IP address
				; A record for the main DNS
				ns1     IN      A       192.168.2.4


				; Define MX record for mail
				example.org. IN   MX   10   mail.example.org.


				; Other domains for example.org
				; Create subdomain www - mail - vault
				www     IN      A       192.168.2.4
				mail    IN      A       192.168.2.6
				vault   IN      A       192.168.2.7
				
	Like the forward zone, edit the reverse zone configuration file
	(/etc/bind/zones/reverse.atadomain.io) and populate the following configuration:
	
				;
				; BIND reverse data file for the local loopback interface
				;
				$TTL    604800
				@       IN      SOA     example.org. root.example.org. (
											1         ; Serial
										604800         ; Refresh
										86400         ; Retry
										2419200         ; Expire
										604800 )       ; Negative Cache TTL

				; Name Server Info for server.example.org
				@       IN      NS      server.example.org.


				; Reverse DNS or PTR Record for server.example.org
				; Using the last number of DNS Server IP address: 192.168.2.4
				10      IN      PTR     server.example.org.


				; Reverse DNS or PTR Record for mail.example.org
				; Using the last block IP address: 192.168.2.6
				20      IN      PTR     mail.example.org.

	Checking the configuration:
		command: 
			// Checking the main configuration for BIND
		  >	sudo named-checkconf

			// Checking forward zone forward.example.org
			command:
			>  sudo named-checkzone example.org /etc/bind/zones/forward.example.org
			If configuration is correct we recive similar output:
				zone example.org/IN: loaded serial 2
				OK
			// Checking reverse zone reverse.example.org
			command:
			>  sudo named-checkzone example.org /etc/bind/zones/reverse.example.org
			If configuration is correct we recive similar output:
				zone example.org/IN: loaded serial 1
				OK
	Opening DNS Port with UFW Firewall:
		command: 
		  >	sudo ufw app list
		We should see output:
			Availbe appilations: Bind9, OpenSSH
		command:
		  >	sudo ufw allow Bind9
		Expected output:
				Rule added
				Rule added (v6)
		Checking enabled rules on the UFW firewall:
		command: 
		  >	sudo ufw status
		Output: Bind9 application on the list.
		
	Verifying BIND DNS Server Installation:
	 
		# Checking the domain names
		commands:
		  >	dig @192.168.2.4 www.example.org
		  >	dig @192.168.2.4 mail.example.org
		  >	dig @192.168.2.4 vault.example.org
			
		All addresser should be resolved to the right server IP
		Next, run the command below to verify the MX record for the  domain.
		  >	dig @192.168.2.4 example.org MX
		Output: domain has the MX record
		
	Commands to verify the PTR record or reverse zone for the server IP addresses 
	If BIND installation is successful, each IP address will be resolved
	to the domain name defined on the reverse.example.org configuration.
		  >	dig @192.168.2.4 -x 192.168.2.4
		  >	dig @192.168.2.4 -x 192.168.2.6
	Changing DNS server for my Ubuntu server:
		command:
		  >	sudo nano /etc/resolv.conf:
			add:
			nameserver 192.168.2.4
			
	Save and quit.
			
			
# *5.Installation and configuration MariaDB:*
	commands:
	  >	sudo apt update
	  >	sudo apt install mariadb-server mariadb-client -y
	  >	sudo systemctl status mariadb
	Output: mariadb should be actived
	Configuration to improve the security of the MariaDB database engine:
	commands:
	  >	sudo mysql_secure_installation
		For all the question press Y
	Configure A Password-authenticated Administrative User:
	-login as the root user:
	  >	sudo mariadb -u root -p
	Create a regular user:
		-CREATE USER
		-'adminuser'@'localhost' IDENTIFIED BY 'place_for password_for_adminuser'
	Expected similar output: Query OK, 0 rows affected (0.000 sec)
	grant all privileges to admin_user. This effectively
	assigns all the database root user’s permissions to the user:
	command:
	  >	GRANT ALL PRIVILEGES ON *.* TO 'admin_user'@'localhost';
	To apply changes:
		FLUSH PRIVILEGES;
	Exit the database:
		EXIT;
	Test MariaDB:
	  >	sudo mariadb -u adminuser -p
	To check the existing databases, run the command:
		SHOW DATABASES;
# *6.Installation and configuration apache2*
	command:
	  >	sudo apt update
	  >	sudo apt install apache2
	Before testing Apache, it’s necessary to modify the firewall
	settings to allow outside access to the default web ports:
		command:
		  >	sudo ufw app list
		  >	sudo ufw allow 'Apache'
		  >	sudo ufw status
		The output will provide a list of allowed HTTP traffic
	Checking web server:
	  >	sudo systemctl status apache2
	At this moment we can type http://192.168.2.4 (address ip of my ubuntu server)
	and see the default Ubuntu 22.04 Apache web page;
# *7. Installation and configuration PHP*
	commands:
	  >	sudo apt update
	  >	sudo apt install php libapache2-mod-php php-mysql
		
	Modify the way that Apache serves files. Currently, 
	if a user requests a directory from the server, Apache will first search for a file
	called index.html. To instruct the web server to prefer PHP files over others,
	you can set Apache to search for an index.php file first:
	
	commands: 
	  >	sudo nano /etc/apache2/mods-enabled/dir.conf
	Move the PHP index file to the first position after
	the DirectoryIndex specification, like in the following:
		<IfModule mod_dir.c>
		DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
		</IfModule>
	Reload Apache’s configuration:
	command:
	  >	sudo systemctl reload apache2
	Checking status:
	  >	sudo systemctl status apache2
	
	Creating a Virtual Host for Website:
		commands:
		  >	sudo mkdir /var/www/example.org
	Next, assign ownership of the directory with the $USER environment variable,
	which will reference current system user:
	command:
	  >	sudo chown -R $USER:$USER /var/www/example.org	
	Then, open a new configuration file in Apache’s sites-available directory:
	command:
	  >	sudo nano /etc/apache2/sites-available/example.org.conf
	This creates a new blank file. Add in the configuration:
	
		<VirtualHost *:80>
		ServerName example.org
		ServerAlias www.example.org 
		ServerAdmin webmaster@localhost
		DocumentRoot /var/www/example.org
		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined
		</VirtualHost>
		
	Save and close file.
	Use a2ensite to enable this virtual host:
	command:
	  >	sudo a2ensite example.org
	To disable Apache’s default website:
	command: 
	  >	sudo a2dissite 000-default
		
	To make sure your configuration file doesn’t contain syntax errors:
	command:
	  >	sudo apache2ctl configtest
	Reload Apache so these changes take effect:
	command:
	  >	sudo systemctl reload apache2
	command:
	  >	sudo nano /etc/apache2/sites-available/000-default.conf
	add:
		ServerName serverubuntu
		ServerAlias www.serverubuntu
	command:
	  >	sudo nano /etc/apache2/apache2.conf
	Add in last line:
		ServerName 192.168.2.4
	Close and save.
	
	command:
	  >	sudo systemctl reload apache2
		
	Testing PHP Processing on your Web Server:
	command: 
	  >	sudo nano /var/www/example.org/info.php
	Add:
		<?php
		phpinfo();
	Save and close.
	*Glpi:*
	command:
	  >	sudo nano /etc/apache2/sites-available/glpi.conf
	add:
	<VirtualHost *:80>
		ServerName 192.168.2.4
		DocumentRoot /var/www/html/glpi
	<Directory /var/www/html/glpi>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
	</Directory>

    ErrorLog ${APACHE_LOG_DIR}/glpi_error.log
    CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
	</VirtualHost>
	
	Save and close.
	commans:
	  >	sudo a2ensite glpi.conf
	  >	sudo systemctl restart apache2
	Now we can finish installation typing http://192.168.2.4 in web browser.
		command:
		  >	sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
		add:
			bind-address 192.168.2.4


		
	
# *8.Weekly backup for DHCP DNS MariaDB and apache2:*
	Create a backup directory:
	  >	sudo mkdir /backup
	After navigate to the directory /backup create a backup script file:
	command:
	  >	sudo nano backup_script.sh
		add: 
			//!/bin/bash

			// DHCP Server Configuration Backup
			cp /etc/dhcp/dhcpd.conf ./dhcpd.conf

			// DNS Server Configuration Backup
			cp /etc/bind/named.conf.options ./named.conf.options
			cp -R /etc/bind/zones ./zones

			// MariaDB Configuration Backup
			mysqldump --defaults-file=/etc/mysql/mariadb.conf.d/mysqld.cnf --all-databases > ./mariadb_backup.sql

			// Apache2 Configuration Backup
			cp -R /etc/apache2 ./apache2
			
			// PHP Files Backup
			cp -R /var/www/html ./php_files

			// Compress the backup files
			tar -czvf backup_files_$(date +%Y%m%d).tar.gz ./*.conf ./zones ./mariadb_backup.sql ./apache2

			// Remove the individual backup files except Apache2
			rm ./dhcpd.conf ./named.conf.options ./mariadb_backup.sql
			rm -rf ./zones ./apache2 ./php_files
			
		Save the file and exit. Make the backup script executable:
		  >	sudo chmod +x backup_script.sh
		  
		Set up a weekly cron job to execute the backup script:
			crontab -e
		Add the line to the crontab file:
			0 0 * * 0 /path/to/backup/directory/backup_script.sh
			
		Save the crontab file and exit.
		
		Now, every week the backup_script.sh will run automatically,
		creating a compressed archive containing the configuration
		files for DHCP server, DNS server, apache2 and MariaDB database dump. 
		The archive file will be saved in the backup directory with the date appended to the filename.
		
# *9. Configuration Client machine Kali Linux:*
	command: 
	  >	sudo nano /etc/network/interfaces
	Configuration the interface should look like:
		auto eth0
		iface eth0 inet dhcp 
	To install LibreOffice, GIMP, and Mullvad browser on Kali Linux, I used the following commands:
	
	*LibreOffice:*
	  >	sudo apt update
	  >	sudo apt install libreoffice
	*GIMP:*
	  >	sudo apt update
	  >	sudo apt install gimp
	*Mullvad browser (Assuming it's available in the package repository):*
	  >	sudo apt update
	  >	sudo apt install mullvad

At this point everything should work properly. 
The project is done.
		
