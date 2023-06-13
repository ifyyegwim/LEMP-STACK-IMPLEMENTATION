# LEMP STACK IMPLEMENTATION IN AWS
A LEMP stack is a bundle of four different software technologies that developers use to build websites and web applications. LEMP is an acronym for the operating system, Linux; the web server, Nginx; the database server, MySQL; and the programming language, PHP. Before attempting this project, I had already created an AWS account and launched an instance running Ubuntu Server OS. I will now descibe the steps taken to succesfully implement LEMP stack.

**STEP 1 - SET UP AN SSH AND CONNECT TO UBUNTU SERVER ON AWS VIA TERMINAL**

<img width="569" alt="Screenshot 2023-06-03 at 09 53 27" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/d51f51f5-5c1e-42db-9b9e-02b877a2640f">


**STEP 2 - INSTALL NGINX WEB SERVER**

In order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server. We’ll use the apt package manager to install this package.

*First, we update a list of packages in package manager with the command:*

    sudo apt update
    
*Next, we run Nginx package installation with the command:*

    sudo apt install nginx

*Then we verify that Nginx is running as a service in our OS with:*

    sudo systemctl status nginx
  
<img width="561" alt="Screenshot 2023-06-02 at 22 45 05" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/ea588f96-fc3e-4412-9863-8eb3694c2c44">

Nginx has been installed successfully.

*We need to add a rule in our EC2 configuration to open inbound configuration through port 80 and set the source to 0.0.0.0/0 so our Web Server can receive traffic and can be accessed from the Internet.*

<img width="1177" alt="Screenshot 2023-06-03 at 09 25 39" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/d0ee3452-60ac-4e13-a56e-003d7b87ed55">

*To confirm we can access our Web Server locally, we run:*

     $ curl http://localhost:80
                   or
     $ curl http://127.0.0.1:80
     
<img width="554" alt="Screenshot 2023-06-02 at 22 46 48" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/290104ce-ae08-4f44-b0c9-fbd1aeee1591">

*To confirm that our Nginx server can respond to requests from the Internet, on a browser, we will try to access the url below*

     http://<Public-IP-Address>:80

<img width="1097" alt="Screenshot 2023-06-02 at 22 48 37" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/0538425a-7796-4b86-a83d-dcd90dad85b6">

Our web server, Nginx, has been installed and is now running.

**STEP 3 - INSTALL MySQL DATABASE SERVER**

*To acquire and install this software, run:*

    sudo apt install mysql-server

*After installation is completed, log in to the MySQL console with:*

    sudo mysql

*Set a password for the root user using mysql_native_password as default authentication method. We're defining this user's password as newpassword. To do this, run*

    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'newpassword';

*Exit the MySQL shell with:*

    exit

*Run a security script to remove insecure default settings and lock down access to your database system using:*

    sudo mysql_secure_installation

*Validate password plugin, remove anonymous users, disable remote root logins e.t.c then log back into the MySQL console using:*

    sudo mysql -p

<img width="552" alt="Screenshot 2023-06-02 at 22 51 57" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/b8cbe32a-b73a-4172-b50a-dcc64227a6f8">

*Exit the MySQL console using:*

    exit

My MySQL server is now installed and secured.

**STEP 4 - INSTALL PHP**

We need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. We also need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases.

*To install the two packages at once, run*

    sudo apt install php-fpm php-mysql
    
PHP has been installed.

**STEP 5 - CONFIGURE NGINX TO USE PHP PROCESSOR**

When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. On Ubuntu 22.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites. I will be using projectLEMP as an example domain name.

*Create the root web directory for projectLEMP by running:*

    sudo mkdir /var/www/projectLEMP

*Assign ownwership of the directory to your current system with the command:*

    sudo chown -R $USER:$USER /var/www/projectLEMP
    
*Created and open a new configuration file in Nginx's sites-available directory using vi with the command:*

    sudo nano /etc/nginx/sites-available/projectLEMP
  
*In the blank file that was created, paste in the following bare-bones configuration:*

    #/etc/nginx/sites-available/projectLEMP

    server {
        listen 80;
        server_name projectLEMP www.projectLEMP;
        root /var/www/projectLEMP;

        index index.html index.htm index.php;

        location / {
        try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        }

        location ~ /\.ht {
        deny all;
        }

    }

Here’s what each of these directives and location blocks do:

listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.

root — Defines the document root where the files served by this website are stored.

index — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.

server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.

location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.

location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.

location ~ /\.ht — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.

Save and close the file by typing **CTRL+X** and then **y** and **ENTER** to confirm.

*Activated your configuration by linking to the config file from Nginx’s sites-enabled directory:*

    sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
    
*Tested your configuration for syntax errors by typing:*

    sudo nginx -t
    
<img width="551" alt="Screenshot 2023-06-02 at 22 55 48" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/804efe7a-ba06-4106-98e6-b765c9193d15">

*Disable the default Nginx host that is currently configured to listen on port 80:* 

    sudo unlink /etc/nginx/sites-enabled/default
    
*Reloaded Nginx to apply changes*C

    sudo systemctl reload nginx
    
*Our new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that the new server block works as expected:*

    sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s         http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
    
*On your browser, try to access the URL:*

    http://<Public-IP-Address>:80
    
<img width="1063" alt="Screenshot 2023-06-02 at 22 56 39" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/1e1c1caf-e64d-4491-a3fd-17bef55cddcc">

**STEP 6 - TESTING PHP WITH NGINX**

*To validate that Nginx can correctly hand .php files off to your PHP processor, open a new file called info.php within your document root in your text editor with the command:*

    sudo nano /var/www/projectLEMP/info.php
    
*Paste into the file that was created, the code:* 

    <?php
    phpinfo();
    
This page can now be accessed in a web browser by visiting the domain name or public IP address you set up in your Nginx configuration file, followed by /info.php:

    http://`server_domain_or_IP`/info.php

    
<img width="1231" alt="Screenshot 2023-06-02 at 22 58 05" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/fcb76eb9-6150-4956-9cd8-d57fb520c18b">

*Remove the file as it contains sensitive information about your PHP environment and your Ubuntu server using:*

    sudo rm /var/www/projectLEMP/info.php
    
This file can be regenerated if necessary.

**STEP 7 - RETRIEVING DATA FROM MySQL USING PHP

Create a test database (DB) with simple “To do list” and configure access to it, so the Nginx website would be able to query data from the DB and display it. We will create a database named example_database and a user named example_user.

*Start by connecting to the MySQL console using the root account:*

    sudo mysql -p
    
*Create a new database:*

    CREATE DATABASE `example_database`;
    
*Create a new user named example_user, using mysql_native_password as default authentication method. We're defining this user’s password as 'newpassword':*

    CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'newpassword';
    
*Give this user permission over the example_database database by running:*

    GRANT ALL ON example_database.* TO 'example_user'@'%';
    
*Exit the MySQL shell with:*

    exit
  
*Test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:*

    mysql -u example_user -p  
    
*To confirm we have access to the example_database database, run:*

    SHOW DATABASES;
    
<img width="691" alt="Screenshot 2023-06-03 at 13 14 16" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/3dde4a1d-fd59-4699-baf0-39ae146a69ce">

*Create a test table named todo_list by running:*

    CREATE TABLE example_database.todo_list (item_id INT AUTO_INCREMENT,content VARCHAR(255),PRIMARY KEY(item_id)); 
    
*Inserte a few rows of content in the test table. Repeat the command below a few times, using different VALUES each time:*

    INSERT INTO example_database.todo_list (content) VALUES ("My first important item");   
  
*To confirm that the data was successfully saved to the table, run:*

    SELECT * FROM example_database.todo_list;  
    
<img width="559" alt="Screenshot 2023-06-02 at 23 47 41" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/c9970f22-70be-43d1-92e0-94d711459348">

*Exit the MySQL console using:*

    exit
    
*Creat a PHP script that will connect to MySQL and query for content. Create a new PHP file in your custom web root directory using vi:*

    nano /var/www/projectLEMP/todo_list.php 
    
The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.  

*Into my todo_list.php script, I pasted:*

     <?php
     $user = "example_user";
     $password = "newpassword";
     $database = "example_database";
     $table = "todo_list";

     try {
       $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
       echo "<h2>TODO</h2><ol>";
       foreach($db->query("SELECT content FROM $table") as $row) {
         echo "<li>" . $row['content'] . "</li>";
       }
       echo "</ol>";
     } catch (PDOException $e) {
          print "Error!: " . $e->getMessage() . "<br/>";
          die();
     }
     
Save and close the file by typing **CTRL+X** and then **y** and **ENTER** to confirm.

*We can accessed this page on my web browser by visiting the domain name or public IP address configured for my website, followed by /todo_list.php:*

<img width="1142" alt="Screenshot 2023-06-02 at 23 49 53" src="https://github.com/ifyyegwim/Breaking-into-DevOps/assets/134213051/bec3a0ec-e075-4905-898e-07803c17f350">


**LEMP STACK HAS BEEN SUCCESSFULLY IMPLEMENTED IN AWS** 🥳

