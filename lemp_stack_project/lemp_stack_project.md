### LEMP STACK PROJECT

### 5/16/2024

### WEB STACK IMPLEMENTATION (LEMP STACK) IN AWS

#### First we start up our instance and ssh into our instance  from our git bash terminal using our .pem file.

Now, to SSH into your instance:

1)Open your terminal (git bash) or use an SSH client.


2)Navigate to the directory where your private key (.pem) file is located. cd Desktop


3)Use the following command to SSH into your instance: ssh -i xxxxxx.pem ubuntu@(my instance public-ip)


### INSTALLING NGINX IN WEB SERVER

In order to render web-pages to our site visitors, we are going to employ the help our nginx a high performance web server.
we will begin by installing nginx in few easy steps.

1)first we find out the latest versions of the packages and dependencies

       sudo apt update 



2)then we install Nginx

       sudo apt install nginx


      

3)To verify that our system was successfully installed and its running we run the command:
     
       sudo systemctl status nginx


## OUR FIRST ISSUE

## Nginx failed to start

with error message( nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Fri 2024-05-17 08:28:55 UTC; 5min ago
       Docs: man:nginx(8)
    Process: 1781 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 1782 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=1/FAILURE)
        CPU: 26ms

May 17 08:28:54 ip-172-31-43-159 nginx[1782]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
May 17 08:28:54 ip-172-31-43-159 nginx[1782]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
May 17 08:28:54 ip-172-31-43-159 nginx[1782]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
May 17 08:28:54 ip-172-31-43-159 nginx[1782]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
May 17 08:28:55 ip-172-31-43-159 nginx[1782]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
May 17 08:28:55 ip-172-31-43-159 nginx[1782]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
May 17 08:28:55 ip-172-31-43-159 nginx[1782]: nginx: [emerg] still could not bind()
May 17 08:28:55 ip-172-31-43-159 systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
May 17 08:28:55 ip-172-31-43-159 systemd[1]: nginx.service: Failed with result 'exit-code'.
May 17 08:28:55 ip-172-31-43-159 systemd[1]: Failed to start nginx.service - A high performance web server and a revers>
lines 1-18/18 (END))

#### solution
our issue came up because we already have a process bound to the HTTP port 80
(Specially after upgrading systems! it will start apache2 by default)

so first we stopped the apache2 service first with 
   
   sudo service apache2 stop 

then we restarted we restarted nginx service with
    
    sudo systemctl restart nginx

now our server is up and running.
first lets try to see how we can access it locally in our ubuntu shell
    
    curl http://localhost:80


then we run it in our browser to view the default nginx page




### INSTALLING MY SQL

MySQL is a widely used open-source relational database management system (RDBMS) that uses 
structured query language (SQL) for database access and management.
It is known for its reliability, scalability, and ease of use, making it a popular choice for web applications.

first we run the command in our git bash terminal 
   
   sudo apt install mysql-server


once installation is complate we log into our console using

     sudo mysql
    
this will log you into mysql server as the administrative database with the user as root,
which is inferred by the use of sudo when running the command

#### Change root Password
1)first we use 
   sudo mysql to loginto mysql

2)then we use the command below to change the password

   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your password';

now we will start the interactive script by running
    
    sudo mysql_secure_installation



now we are done we will check if we are able to log into my sql using command below

      sudo mysql -p


mysql server is now installed and secured 


### INSTALLING PHP

To install PHP on an EC2 instance with Nginx already running, first connect to your instance via SSH. 
Update your package lists using sudo apt-get update (for Ubuntu) or sudo yum update (for Amazon Linux). 
Install PHP and necessary PHP-FPM packages with sudo apt-get install php-fpm (for Ubuntu) or sudo yum install php-fpm (for Amazon Linux). 
Configure Nginx to use PHP by editing the Nginx configuration file, typically found at /etc/nginx/sites-available/default, to include a location block for PHP files. 
Restart Nginx with sudo systemctl restart nginx and start PHP-FPM with sudo systemctl start php-fpm.

PHP-FPM enhances the performance and reliability of PHP-based web applications running alongside Nginx.

    sudo apt install php-fpm php-mysql



Next we will configure nginx to use them

#### Configuring Nginx to use php processor

Using the nginx server you can create server blocks similar to virtual hosts in apache2 to encapsulate configuration details and host more than one domanin 
on a single server.
Nginx has one server block enabled by default and its configured to serve documents out of a directory at /var/www/html. 
while this works well for a single site it might be confusing and difficult to manage if you are hosting multiple sites. instead of modifying /var/www/html,
we will create a directory strusture within /var/www for your the your_domain website, leaving /
var/www/html in place as the default directory if a clients request does not match any available site.

create root web directory for your_domain as follows:
      
      sudo mkdir /var/www/projectLEMP


next assign ownership of the directory with the $USER enviroment variable,which will reference your current system user:

     sudo chown -R $USER:$USER /var/www/projectLEMP



Open a new configuration file in Nginx's sites-available directory. here we will use nano

     sudo nano /etc/nginx/sites-available/projectLEMP


now we will type  the bare bones in the blank file



activate your configuration by linking to the config file nginx's sites-available directory:
   
   sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/


test configuration systext error by typingn:

    sudo nginx -t



 we need to disable the default nginx host that is currently configuired to listen to port 80.
 for this run:

       sudo unlink /etc/nginx/sites-enabled/default


reload nginx to apply changes:

      sudo systemctl reload nginx


new website is now active but the web root /var/www/projectLEMP is still empty , so we create an index.html file in that location so we can test 
that our new server block works as expected:
      
      sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/
      latest/meta-data/public-hostname) 'with public IP' $(curl -s http://
      169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/
      index.html

issue that came up:

           -bash: latest/meta-data/public-hostname: No such file or directory
           -bash: 169.254.169.254/latest/meta-data/public-ipv4: No such file or directory
           -bash: /var/www/projectLEMP/: Is a directory
           index.html: command not found





solution:
        

       sudo bash -c 'echo "Hello LEMP from hostname $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) with public IP $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)" > /var/www/projectLEMP/index.html'
 
       ![alt text]()



Lemp stack is now fully configured 


### testing php with nginx

we can test to validate that nginx can properly handle .php files off to php processor.
we can do this by creating a test php file in our document root. Open a new file called info.php within your document root in your text editor, here we are using nano text editor
      

      nano /var/www/projectLEMP/info.php


type the following lines in the new file,this is a valid php code that will return information about your server

     <?php
     phpinfo();


Experienced issue with 502 bad gateway and realized my file edited with sudo nano /etc/nginx/sites-available/projectLEMP was not formatted properly.




now we will remove the file as it contains sensitive information

     sudo rm /var/www/projectLEMP/info.php


### RETRIEVING DATA FROM MYSQL DATABADE

we will create a to-do list data base and configure access to it via mysql

first connect to mysql
   sudo mysql -u root -p

create a new database running the following command

    CREATE DATABASE example_database;


create a new user and grant him full access on the database:
   
   CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY '*******';


now we need to give user permission over example_database database

    GRANT ALL ON example_database.* TO 'example_user'@'%';


now we test if the new user has the permissions granted by logging into database again:
   

   mysql -u example_user -p

now we want to view the database
    
    SHOW DATABASES;



Now we will create table named todo_list

     CREATE TABLE example_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY (item_id)
);

insert a few content
   mysql> INSERT INTO example_database.todo_list (content) VALUES ("My
   first important item");


confirm data was successfully saved run command below:

   SELECT * FROM example_database.todo_list;



after that exit my sql


Now we create a php script that will connect to mysql and query it to be rendered on our website:
create a new php file in your cutom web root directory using your preferred editor:

     sudo nano /var/www/projectLEMP/todo_list.php



copy and paste content below into your file:

    <?php
     $user = "example_user";
     $password = "xxxxxxxx";
     $database = "example_database";
     $table = "todo_list";
     try {
     $db = new PDO("mysql:host=localhost;dbname=$database", $user,
     $password);
     echo "<h2>TODO</h2><ol>";
     foreach($db->query("SELECT content FROM $table") as $row) {
     echo "<li>" . $row['content'] . "</li>";
     }
     echo "</ol>";
     } catch (PDOException $e) {
     print "Error!: " . $e->getMessage() . "<br/>";
     die();
     }


save and close now visit your public domain ending with /todo_list.php
