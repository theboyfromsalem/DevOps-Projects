## **IMPLEMENTATION OF A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS).**

### CLIENT-SERVER ARCHITECTURE WITH MYSQL

Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another. In their communication, each machine has its own role: the machine sending requests is usually referred as "Client" and the machine responding (serving) is called "Server". A simple diagram of Web Client-Server architecture is presented below:

![alt text](<Images/illustration 1.png>)

In the example above, a machine that is trying to access a Web site using a Web browser or simply ‘curl’ command is a client and it sends HTTP requests to a Web server (Apache, Nginx, IIS or any other) over the Internet. If we extend this concept further and add a Database Server to our architecture, we can get this picture:

![alt text](<Images/illustration 2.png>)

The Web Server has a role of a "Client" that connects and reads/writes to/from a Database (DB) Server (MySQL, MongoDB, Oracle, SQL Server or any other), and the communication between them happens over a Local Network (it can also be an Internet connection, but it is a common practice to place Web Server and DB Server close to each other in a local network).
Essentially, it is sending requests to the remote server, and in turn, would be expecting some kind of response from the remote server.

### **TO DEMONSTRATE A BASIC CLIENT-SERVER USING MYSQL RELATIONAL DATABASE MANAGEMENT SYSTEM (RDBMS), FOLLOW THE BELOW INSTRUCTIONS:**

-  Create and configure two Linux-based virtual servers (EC2 instances in AWS).
`Server A name - mysql-server` and `Server B name - mysql-client`

![alt text](<Images/Screenshot 2026-02-25 181551.png>)

Let’s take a very quick example and see Client-Server communicatation in action. Open up your Ubuntu or Windows terminal and run the curl command:

`curl -Iv www.bing.com`

***Note: If your Ubuntu does not have ‘curl’, you can install it by running `sudo apt install curl` In this example, your terminal will be the client, while www.bing.com will be the server.***

See the response from the remote server in the below output. You can also see that the requests from the URL are being served by a computer with an IP address `2.18.67.160` on port 80.

![alt text](<Images/Screenshot 2026-02-25 182726.png>)

- Now, let's implement this using our created instances. On mysql-server Linux Server install MySQL Server software.
`sudo apt update`

- Then install the mysql-server package:
`sudo apt install mysql-server`

- Ensure that the server is running using the systemctl command:
`sudo systemctl start mysql.service`                  `sudo systemctl status mysql.service`

![alt text](<Images/Screenshot 2026-02-26 022425.png>)

### SETTING IT UP

`sudo mysql`

For the password I'll be using 'Password'

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord';`

![alt text](<Images/Screenshot 2026-02-26 022945.png>)

Run a MySQL secure installation
`sudo mysql_secure_installation`

![alt text](<Images/Screenshot 2026-02-26 023913.png>)

Answer Y for yes, or anything else to continue without enabling.

- In the MySQL server create a user and a database named first_db and a user named first_user, but you can replace these names with different values.

- First, connect to the MySQL console using the root account:
`sudo mysql -p`

- Create a new database by running this command from your MySQL console:
`CREATE DATABASE example_database;`

Create a new user and grant full privileges on the database we have just created.
`CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord';`

***Note: The following command above creates a new user named example_user, using mysql_native_password as default authentication method. We’re defining this user’s password as password, but you should replace this value with a secure password of your own choosing.***

 **TROUBLESHOOT:** I encountered this errror ***"ERROR 1819 (HY000): Your password does not satisfy the current policy requirements"***. This error happens because the MySQL Validate Password Component is active and my chosen password is too simple for its current security settings. So I ran the following commands to resolve `SHOW VARIABLES LIKE 'validate_password%';`, `SET GLOBAL validate_password.policy = LOW;`, 
`SET GLOBAL validate_password.length = 4;`

![alt text](<Images/Screenshot 2026-02-26 025645.png>)

![alt text](<Images/Screenshot 2026-02-26 025351.png>)

![alt text](<Images/Screenshot 2026-02-26 025507.png>)

- Give this user permission over the example_database database:
`GRANT ALL ON example_database.* TO 'example_user'@'%';`

![alt text](<Images/Screenshot 2026-02-26 031703.png>)

***Note: This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.***

Exit the MySQL shell with: `exit`

- Test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:
`mysql -u example_user -p`

***The -p flag in this command, which will prompt us for the password used when creating the example_user user.***

- After logging in to the MySQL console, confirm that you have access to the example_database database: mysql> `SHOW DATABASES;`
This will give you the following output highlighted in green :

![alt text](<Images/Screenshot 2026-02-26 032349.png>)

- Exit MySQL, then restart the mySQL service using:

`sudo systemctl restart mysql`, `sudo systemctl status mysql.service`

![alt text](<Images/Screenshot 2026-02-26 033209.png>)

- You might need to configure MySQL server to allow connections from remote hosts.

- `sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf`

![alt text](<Images/Screenshot 2026-02-26 033435.png>)

- By default, both of your EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups.

![alt text](<Images/Screenshot 2026-02-26 034301.png>)

Mysql client private ip address is used above instead of 0.0.0.0 for extra Security

- Save the above configurations.

### SET UP MYSQL CLIENT


- ssh into mysql-client instance

- On mysql client Linux Server install MySQL client software
`sudo apt update && sudo apt ugrade`

- Install the mysql-client package: `sudo apt install mysql-client -y`

![alt text](<Images/Screenshot 2026-03-01 011950.png>)

From mysql client instance connect remotely to mysql server Database using:
`sudo mysql -u example_user -h <mysqlserver private ip> -p`

When I ran this command I encountered an error message `ERROR 2003 (HY000): Can't connect to MySQL server on '172.31.21.5:3306' (111)`

**TROUBLESHOOT**
I did a lot of trials to figure out why it was throwing that error, here are some of the area I looked at:

- I checked the service layer because I felt the MySQL service wasn't installed correctly or was inactive. I performed a clean install on the Server, ensuring the software was actually present and running in the background.

![alt text](<Images/Screenshot 2026-03-01 015428.png>)

- I looked at the authentication layer, I created a user with the `%` wildcard (``'koffebwoy'@'%'`). This told the database: "I trust this user, no matter what machine they are calling from.

![alt text](<Images/Screenshot 2026-03-01 015746.png>)

- I looked at the configuration layer, I felt it only listened to itself, I changed the `bind-address` to `0.0.0.0` in the config. file.

Finally, I edited the inbound rules in the AWS cosole, and I changed it from my IP to IPV4`(0.0.0.0)` and that was the magic.

![alt text](<Images/Screenshot 2026-03-01 015008.png>)

I tried the command `sudo mysql -u example_user -h <mysqlserver private ip> -p` again and it connected.

![alt text](image.png)