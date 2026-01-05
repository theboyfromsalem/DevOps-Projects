# LAMP STACK IMPLEMENTATION (Linux, Apache, MySQL and PHP)


## Step 0 - Preparing Prerequisite

- Create an AWS account.
![alt text](<Images/Screenshot 2026-01-05 030142.png>)

- Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM) on AWS.
![alt text](<Images/Screenshot 2026-01-05 033854.png>)
- Connect to the instance on your local machine.
![alt text](<Images/Screenshot 2026-01-05 034825.png>)

![alt text](<Images/Screenshot 2026-01-05 035241.png>)

![alt text](<Images/Screenshot 2026-01-05 040704.png>)


![alt text](<Images/Screenshot 2026-01-05 041023.png>)


## Step 1 - INSTALLING APACHE AND UPDATING THE FIREWALL

- Install Apache using Ubuntu’s package manager `apt`
- Update a list of packages in package manager
`sudo apt update` 
- Run apache2 package installation
`sudo apt install apache2`

![alt text](<Images/Screenshot 2026-01-01 230517.png>)

- To verify that apache2 is running as a Service in our OS, use following command
 `sudo systemctl status apache2`

- Open inbound connection through port 80
![alt text](<Images/Screenshot 2026-01-01 231434.png>)

- To view the Apache default page, Go to http://Public-IP-Address:80 in your browser

![alt text](<Images/Screenshot 2026-01-01 233242.png>)


## Step 2 - INSTALLING MYSQL

- Again, use `apt` to acquire and install this software:   
`sudo apt install mysql-server`

- When prompted, confirm installation by typing Y, and then ENTER.

- When the installation is finished, log in to the MySQL console by typing:     
`sudo mysql`
- This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command.
- Define the users password using this command:    
`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord'`
- Exit the MySQL shell with:    
`mysql> exit`
- Start the interactive script by running:  
`sudo mysql_secure_installation`

- This will ask if you want to configure the VALIDATE PASSWORD PLUGIN.
- Answer Y for yes, or anything else to continue without enabling 
![alt text](<Images/Screenshot 2026-01-02 000322.png>)

- When you’re finished, test if you’re able to log in to the MySQL console by typing:   
`sudo mysql -p`
- To exit the MySQL console, type:
   `mysql> exit`

## Step 3 — INSTALLING PHP
 - To install these 3 packages at once, run:  
`sudo apt install php libapache2-mod-php php-mysql`
![alt text](<Images/Screenshot 2026-01-02 003007.png>)
- Once the installation is finished, you can run the following command to confirm your PHP version:   
`php -v`

- At this point, your LAMP stack is completely installed and fully operational.

      +Linux (Ubuntu)
      +Apache 
      +HTTP Server
      +MySQL
      +PHP


## Step 4 — CREATING A VIRTUAL HOST FOR YOUR WEBSITE USING APACHE 

- Create the directory for 'newproject' using mkdir command as follows:   
`sudo mkdir /var/www/newproject`
- Next, assign ownership of the directory with your current system user:  
  `sudo chown -R $USER:$USER /var/www/newproject`
- Create and open a new configuration file in Apache’s sites-available directory using:
 `sudo nano /etc/apache2/sites-available/newproject.conf`
- Paste in the following bare-bones configuration


  `<VirtualHost *:80>
ServerName newproject
ServerAlias www.newproject
ServerAdmin webmaster@localhost
DocumentRoot /var/www/newproject
ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>`

- You can use the `ls` command to show the new file in the sites-available directory:                           
`sudo ls /etc/apache2/sites-available`

- You can now use `a2ensite` command to enable the new virtual host:    
`sudo a2ensite newproject`
-  To disable Apache’s default website use `a2dissite` command, type:
`sudo a2dissite 000-default`
- To make sure your configuration file doesn’t contain syntax errors, run:  
`sudo apache2ctl configtest`

- Finally, reload Apache so these changes take effect:  
`sudo systemctl reload apache2`
![alt text](<Images/Screenshot 2026-01-02 010801.png>)


## Step 5 — ENABLE PHP ON THE WEBSITE

- With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php file.

- In case you want to change this behavior, you’ll need to edit the /etc/apache2/mods-enabled/dir.conf file and change the order in which the index.php file is listed within the DirectoryIndex directive:

`sudo nano /etc/apache2/mods-enabled/dir.conf`

`<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
`

- After saving and closing the file, you will need to reload Apache so the changes take effect:

`sudo systemctl reload apache2`

![alt text](<Images/Screenshot 2026-01-02 013518.png>)


- Finally, we will create a PHP script to test that PHP is correctly installed and configured on your server.

- Create a new file named index.php inside your custom web root folder:

`vim /var/www/newproject/index.php`

- This will open a blank file. Add the following text, which is valid PHP code, inside the file:

`<?php
phpinfo();
`

- When you are finished, save and close the file, refresh the page and you will see a page similar to this:
![alt text](Images/php-page.png)













