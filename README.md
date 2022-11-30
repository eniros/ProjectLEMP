# ProjectLEMP
Implementation of LEMP Stacks

This demonstrates the implementation of a web (LEMP) stack in aws.

STEP 1 — INSTALLING THE NGINX WEB SERVER

Install Nginx using the following commands:

#update a list of packages in package manager:
sudo apt update
 
#run Nginx package installation:
sudo install nginx


 ![installnginxsc](https://user-images.githubusercontent.com/61475969/204671313-b05b0372-ffca-424c-9b8f-a69148982b10.png)

Verify your installation by running the following command:
sudo systemctl status nginx
 
 ![nginxstatuscheck](https://user-images.githubusercontent.com/61475969/204671487-38ad138a-af25-4f92-9a40-74931bb958e8.png)
 
 Add a new inbound rule to the EC2 Instance's firewall if you have not already added in associated VPC; this will allow the EC2 Instance to receive HTTP requests from the Internet. The screenshot below shows my current inbound rule for the associated VPC so there's no need to add inbound rule at instance level.
 
 <img width="1035" alt="Screenshot 2022-11-29 at 23 35 22" src="https://user-images.githubusercontent.com/61475969/204672139-96ca6d99-d68a-47a2-a196-5adf3e01ffb1.png">

Access you new Nginx web server using the following URL:

 http://<ec2-instance-public-ip-address>
  
  ![DefaultNginxpage](https://user-images.githubusercontent.com/61475969/204672263-249dada9-95ac-4996-bdc4-5696b192c2ff.png)
  
  NOTE: The EC2 Instance's public IP address can be found by running the following command:

 curl -s http://169.254.169.254/latest/meta-data/public-ipv4
  
STEP 2 — INSTALLING MYSQL AND UPDATING THE FIREWALL

Install MySQL using the following commands:
sudo apt install mysql-server
  
You need to run security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Start the interactive script by running:
sudo mysql_secure_installation
  
  This will ask you to configure the validation plugin which helps with passsword strength check. If one wants it enabaled; answer Y to the prompt If not, press any other key. <\br> NOTE: Always use a strong password for MySQL.
  
  ![mysqlinitalprompt](https://user-images.githubusercontent.com/61475969/204672441-8cc1628c-19a1-49a3-a4a6-c526b83dedab.png)
  
Test your MySQL installation by running the following command:
sudo mysql
  
  If installation is successful, you should see the following message:
  
 ![installationcompletescreen](https://user-images.githubusercontent.com/61475969/204672576-93d839f6-d2c1-411f-a851-9c67f34ed8c5.png)
  
  Note: At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. For that reason, when creating database users for PHP applications on MySQL 8, you’ll need to make sure they’re configured to use mysql_native_password instead.

STEP 3 — INSTALLING PHP

To install these 3 packages at once, run:
sudo apt install php-fpm php-mysql
  
The command above will install the following packages: 
PHP 

php-fpm(PHP fastCGI process manager) to tell Nginx to pass PHP requests to this software for processing

php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases.

Image

STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR

To create a domain (e.g projectlemp) for an app we need to create a server blocks just like virtual host in Apache for the domain. This can be useful if we want to host more than one domain on the server.<\br>

Note: Nginx enables a sever block by default and its configured to serve documents in var/www/html directory. This is great if we are hosting just one website/application. But if we want to host more than one website/application, we need to create a server block for each website/application. In order to achive this,we’ll create a directory structure within /var/www for each website/application we are hosting on the server.
The server block in var/www/html will serve requests that doesnt match any other website. 

Create the root web directory for projectlemp using ‘mkdir’ command as follows:
sudo mkdir /var/www/projectlemp
  
Assign ownership of the directory to current user using ‘chown’ command as follows:
 sudo chown -R $USER:$USER /var/www/projectlemp
  
Create and open a new configuration file in Nginx's sites-available directory using nano editor command as follows:
sudo nano /etc/nginx/sites-available/projectlemp
  
Add the following lines to the file:
#/etc/nginx/sites-available/projectlemp

server {
    listen 80;
    server_name projectlemp www.projectlemp;
    root /var/www/projectlemp;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
Note the following:

location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.
  
location ~ \.php$ — The second location block includes a fastcgi_pass directive, which tells Nginx to pass PHP requests to the PHP-FPM process manager.
  
location ~ /\.ht { — The third location block includes a deny all directive, which tells Nginx to deny all requests to files or directories that begin with a dot. This prevents Nginx from serving files or directories that are hidden or system-specific.
  
Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:
  
  sudo ln -s /etc/nginx/sites-available/projectlemp /etc/nginx/sites-enabled/
  
Test your configuration with the following command:
  
sudo nginx -t
  
Disable the default server block by removing the symbolic link from sites-enabled directory:
  
sudo unlink /etc/nginx/sites-enabled/default   
  
Restart Nginx using the following command:
  
sudo systemctl reload nginx
  
In order to access the website, we need to create an index.html file in the projectlamp directory and output a message.
  
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlemp/index.html
  
Verify the website by visiting:
  
http://<Public-IP-Address>:80

  
![lempindexpage](https://user-images.githubusercontent.com/61475969/204674236-ac942e64-40bc-4c32-af8e-fd395197e6b6.png)
  
  STEP 5 – TESTING PHP WITH NGINX

We need to test if our Nginx can transfer .php files to PHP processor

To do this; we need to open a new file called info.php within your document root in your text editor:

sudo nano /var/www/projectlemp/info.php
Then add the line of code below:

<?php phpinfo(); ?>
We can now access this file at:

http://server_domain_or_IP/info.php

  ![lempinfopage](https://user-images.githubusercontent.com/61475969/204674341-c1f2133d-5734-42cf-928e-0ba1e0501bec.png)
  
  STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)

We will create a database named example_database and a user named example_user, but you can replace these names with different values.

Create a new database named example_database using the following command:
CREATE DATABASE `example_database`;
Create a database User named example_user with the following command:
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
The command above creates a new user named example_user, using mysql_native_password as default authentication method

Assign the user permission over the example_database database:
GRANT ALL PRIVILEGES ON `example_database`.* TO 'example_user'@'%';
Exit the MySQL shell and log back in as the example_user:
exit
Connect to the database using the following command:
mysql -u example_user -p    
Show the list of databases:
show databases;


Lets create a table named todo_list in the example_database database:
CREATE TABLE example_database.todo_list (
item_id INT AUTO_INCREMENT,
content VARCHAR(255),
PRIMARY KEY(item_id)
);
Insert some data into the table:
INSERT INTO example_database.todo_list (content) VALUES ('Buy milk');
INSERT INTO example_database.todo_list (content) VALUES ('Buy eggs');
INSERT INTO example_database.todo_list (content) VALUES ('Buy bread');
Check data in the example_database database:
SELECT * FROM example_database.todo_list;   
 
Create a php script to display the data from the table:
sudo nano /var/www/projectlemp/todo_list.php
 
Add the following code to the file:
 
 
<?php
$user = "example_user";
$password = "password";
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


![sqltable](https://user-images.githubusercontent.com/61475969/204675072-f0b334f5-6133-41e9-8c52-215451745910.png)

Test the php script by visiting:
http://server_domain_or_IP/todo_list.php

![tododatasc](https://user-images.githubusercontent.com/61475969/204675107-6a67decd-a6b9-4feb-b7dc-ad9f07e643f2.png)


