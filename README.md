# LEMP-Stack-Implimentation
Web Stack Implementation Using LEMP
* I already have an account with aws, and I have launched an EC2 instance of t2.micro family with the latest Ubuntu Server 

* Now, I am going to establish and ssh connection with the following command;

```
ssh -i <Your-private-key.pem> ubuntu@<EC2-Public-DNS>

This is how it looks like:

```
![Screen Shot 2023-01-26 at 2 49 59 PM](https://user-images.githubusercontent.com/117458922/214870523-c7877d55-0322-405a-acef-09ca81474bd0.png)

## Step 1 – Installing the Nginx Web Server

* In order to display web pages to our website visitors we are going to make use of Nginx as Web Server. We'll use the apt Package Manager to install this package.

Since this is our first time using apt for this session, we should start of by updating your server's package index. Thereafter, we can use `apt install` to get Nginx installed:

```
sudo apt update
sudo apt install nginx
```

To verify that nginx was successfully installed and running as a service in Ubuntu, run:

```
$ sudo systemctl status nginx
```

![Screen Shot 2023-01-26 at 2 51 30 PM](https://user-images.githubusercontent.com/117458922/214873210-d01e6450-ebae-4d8d-b748-53d1a05f3607.png)

* Note that, if it is green and running, then you did everything correctly - you have just successfully launched your web Server in the Clouds!

However, before we can receive any traffic by our Web Server, we need to open TCP port 80 which is default port that web browsers use to access web pages in the Internet. TCP port 22 is open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80 if you did not add the rule before you launched the EC2 instance.

![security group](https://user-images.githubusercontent.com/117458922/214875136-5e0a44a5-1475-4044-bfd9-ceaeb36bde34.png)

* Now, our server is running and we can access it locally and from the Internet (Source 0.0.0.0/0 means ‘from any IP address’).

  So, let us try to check how we can access it locally in our Ubuntu shell by running this command:

```
$ curl http://localhost:80
or
$ curl http://127.0.0.1:80
```

These 2 commands above actually do pretty much the same

* The output should be some strangely formatted test, do not worry, we just made sure that our Nginx web service responds to ‘curl’ command with some payload.

![curl](https://user-images.githubusercontent.com/117458922/214875935-0171ed91-f76c-4245-97c1-f56bb5855dd4.png)

* Open a web browser of your choice and try to access following url

```
http://<Public-IP-Address>:80
```
Another way you can retrieve your Public IP address, other than to check it in AWS Web console, is to use following command:

```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

The URL in browser shall also work if you do not specify port number since all web browsers use port 80 by default.

If you see the following page, then your web server is now correctly installed and accessible through your firewall.

![nginx](https://user-images.githubusercontent.com/117458922/214877158-eeed0b33-81d7-4901-84aa-b70ebfe20733.png)

And it is in fact the same content that you previously got by ‘curl’ command, but represented in nice HTML format by your web browser.


## Step 2 — Installing MySQL

* Now that the Web Server active and running, you need to install a Database Management System(DBMS) to be able to store and manage data for your site in a relational database. MySQL is a popular relational database management system used within PHP environments, so we will use it in our project. Again use `apt` to acquire and install this software:

```
sudo apt install mysql-server -y
```

* When the installation is finished, log in to the MySQL console by typing:

```
sudo mysql
```

This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command. You should see output like this:
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.22-0ubuntu0.20.04.3 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

* It’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Before running the script you will set a password for the root user, using mysql_native_password as default authentication method. We’re defining this user’s password as PassWord.1.

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

* Exit the MySQL shell with:

```
mysql> exit
```

![Screen Shot 2023-01-26 at 2 59 19 PM](https://user-images.githubusercontent.com/117458922/214880073-1cd90c37-973f-44d4-b386-b3b1ea98df98.png)

* Now, start the interactive script by running:

```
sudo mysql_secure_installation
```

This will ask if you want to configure the VALIDATE PASSWORD PLUGIN. Answer Y for "yes" or anything else to continue without enabling. If you answer "yes", you'll be asked to select a level of password validation. Keep in mind that if you enter 2 for the strongest level you will receive errors when attempting to set any password which does not contain numbers, upper and lowercase letters, and special charachers, or which is based on common dictionary words.

Regardless of whether you chose to set up the VALIDATE PASSWORD PLUGIN, your Server will next ask you to select and confirm a password for MySQL root user. This is not to be confused with the system root. The database root user is an administrative user with full privileges over the database system. Even though the default authentication method for the MySQL root user dispenses the use of a password, even when one is set, you should define a strong password here as an additional safety measure.

If you enabled password validation, you'll be shown the password strength for the root password you just entered and your Server will ask if you want to continue with that password. If you are happy with your current password, enter Y for "yes" at the prompt.

For the rest of the questions, press Y and hit ENTER key at the prompt. This will remove some anonymous users and the test database, disable remote root logins, and load these new rules so that MySQL immediately respect the changes you have made.

When you are finished, test if you're able to log into the MySQL console by typing:

```
sudo mysql -p
```

Notice the -p flag in this command, which will prompt you for the password used after changing the root user password.

![Screen Shot 2023-01-26 at 3 03 47 PM](https://user-images.githubusercontent.com/117458922/214883230-9fc3c76b-bf23-43e8-98c2-ec2d0600ba93.png)


* Note that you need to provide a password to connect as the root user.

For increased security, it’s best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server.

Note: At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. For that reason, when creating database users for PHP applications on MySQL 8, you’ll need to make sure they’re configured to use mysql_native_password instead.

* Now, our MySQL server is installed and secured. Next thing we will do is to install PHP, the final component in the LAMP stack.

## STEP 3 - Installing PHP

We have  installed Nginx to serve our content and MySQL installed to store and manage data. Now we can install PHP to process code and generate dynamic content for the Web Server.

While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the Web Server. This allows for a better performance in most PHP-based websites, but it requires additional configuration. We will need to install php-fpm, which stands for "PHP fastCGI Process Manager", and tell Nginx to pass PHP requests to this software for processing. Additionally we will need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependenes.

To install these 2 packages at once, run this command:

```
sudo apt install php-fpm php-mysql -y
```

## STEP 4 — Configuring Nginx to Use PHP Processor

When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. We will use projectLEMP as an example domain name.

On Ubuntu, Nginx has one server block enabled by default and it is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.


* Create the root web directory for your_domain as follows:

```
sudo mkdir /var/www/projectLEMP
```

* Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:

```
sudo chown -R $USER:$USER /var/www/projectLEMP
```

* Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:

```
sudo nano /etc/nginx/sites-available/projectLEMP
```

This will create a new blank file. Paste in the following bare-bones configuration:

```
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
```

![Screen Shot 2023-01-26 at 3 09 02 PM](https://user-images.githubusercontent.com/117458922/214886690-b1c5af51-6b61-4a44-b478-7c141d23ec3f.png)

* Here’s what each of these directives and location blocks do:

listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.

root — Defines the document root where the files served by this website are stored.

index — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.

server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.

location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.

location ~ .php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.

location ~ /.ht — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.

When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.
* To save and close *Nano* use `CTL + O` then `Enter` and `CTL + X` to close

Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:

```
sudo ln -s /etc//nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:

```
sudo nginx -t
```

You shall see following message:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

![Screen Shot 2023-01-26 at 3 09 25 PM](https://user-images.githubusercontent.com/117458922/214888084-8f354fc3-97c1-44fc-8142-611207dfda93.png)

If any errors are reported, go back to your configuration file to review its contents before continuing.

We also need to disable default Nginx host that is currently configured to listen on port 80, for this run:

```
sudo unlink /etc/nginx/sites-enabled/default
```

When you are ready, reload Nginx to apply the changes:

```
sudo systemctl reload nginx
```

Now, our new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:

Use the following command;

```
sudo echo 'Hello LEMP, I am Kolawole from UltraCloud' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' 
(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```

Now let us go to our browser and try to open the website URL using IP address:

```
http://<Public-DNS-Name>:80
```

You can also access your website in your browser by public DNS name, not only by IP - try it out, the result must be the same (port is optional)

```
http://<Public-DNS-Name>:80
```

You should see something like this;

![Screen Shot 2023-01-26 at 3 17 30 PM](https://user-images.githubusercontent.com/117458922/214889554-08e4b61e-7d6a-49ee-ab0c-6afe780d5c52.png)

We can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.

Our LEMP stack is now fully configured. In the next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.


## Step 5 – Testing PHP with Nginx

Our LEMP stack should now be completely set up.

At this point, your LEMP stack is completely installed and fully operational.

You can test it to validate that Nginx can correctly hand .php files off to your PHP processor.

You can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:

```
$ nano /var/www/projectLEMP/info.php
```

Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:

```
<?php
phpinfo();
```

We can now access this page in our web browser by visiting the domain name or public IP address we set up in our Nginx configuration file, followed by /info.php:

```
http://`server_domain_or_IP`/info.php
```
You will see a web page containing detailed information about your server:

![Screen Shot 2023-01-26 at 3 27 18 PM](https://user-images.githubusercontent.com/117458922/214948230-673a7aed-3228-4270-be85-63ff5f640616.png)

After checking the relevant information about our PHP server through that page, it’s best to remove the file we created as it contains sensitive information about the PHP environment and the Ubuntu server. Use `rm` to remove that file:

```
$ sudo rm /var/www/your_domain/info.php
```

This file can always be regenerated if there is need for it later.

## STEP 6 - Retrieving data from MySQL database with PHP

In this step, we will create a test database (DB) with simple “To do list” and configure access to it, so that the Nginx website would be able to query data from the DB and display it.

At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. We’ll need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.

We will create a database named example_database and a user named example_user, but you can replace these names with different values if you wish.

First, we will connect to the MySQL console using the root account:

```
$ sudo mysql -p
```

To create a new database, we run the following command from our MySQL console:

```
mysql> CREATE DATABASE `example_database`;
```

Now we can create a new user and grant him full privileges on the database you have just created.

The following command creates a new user named example_user, using mysql_native_password as default authentication method. We’re defining this user’s password as password, but you should replace this value with a secure password of your own choosing.

```
mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```

Now we need to give this user permission over the example_database database:

```
mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';
```
This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.

Now exit the MySQL shell with:

```
mysql> exit
```

![Screen Shot 2023-01-26 at 3 30 29 PM](https://user-images.githubusercontent.com/117458922/215123250-fa059b5f-4d5f-45b4-a86b-da9397751f99.png)

We can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:

```
$ mysql -u example_user -p
```

Notice the -p flag in this command, which will prompt for the password used when creating the example_user user. After logging in to the MySQL console, we should confirm that you have access to the example_database database:

```
mysql> SHOW DATABASES;
```

This gives us the following output:

![Screen Shot 2023-01-26 at 3 32 11 PM](https://user-images.githubusercontent.com/117458922/215123919-9885ef39-a642-49af-abbc-595f6cfe4f5b.png)

Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:

CREATE TABLE example_database.todo_list (

```
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
```

![Screen Shot 2023-01-26 at 3 42 15 PM](https://user-images.githubusercontent.com/117458922/215124800-2c28b93a-7c95-456b-8c12-57e62e104a8a.png)

* Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different VALUES:

```
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
```

![Screen Shot 2023-01-26 at 3 43 59 PM](https://user-images.githubusercontent.com/117458922/215124887-c8424807-35de-41b9-b13f-84db213aafbd.png)


To confirm that the data was successfully saved to your table, run:

```
mysql>  SELECT * FROM example_database.todo_list;
```

You’ll see the following output:

![Screen Shot 2023-01-26 at 3 44 42 PM](https://user-images.githubusercontent.com/117458922/215125459-c519341f-d231-41d4-a1ef-5202057f2b6f.png)

 After confirming that you have valid data in your test table, you can exit the MySQL console:

```
mysql> exit
```

* Create a new PHP file in your custom web root directory using your preferred editor. We’ll use vi for that:


```
$ nano /var/www/projectLEMP/todo_list.php
```

The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.

We will copy this content into our todo_list.php script:

```
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
```

![Screen Shot 2023-01-26 at 3 45 47 PM](https://user-images.githubusercontent.com/117458922/215125963-c54b00f2-e2b6-4bef-a2f2-a6757299df71.png)

We can now access this page in our web browser by visiting the domain name or public IP address configured for our website or by using the default localhost, followed by `/todo_list.php.`

```
http://localhost/todo_list.php
```

You should see a page like this, showing the content we inserted in our test table:

![Screen Shot 2023-01-26 at 3 47 37 PM](https://user-images.githubusercontent.com/117458922/215126349-6ee49c19-376e-4531-916c-c032e8689cb1.png)

It means our PHP environment is ready to connect and interact with our MySQL server.

Congratulations!

We have built a flexible foundation for serving PHP websites and applications to our visitors, using Nginx as web server and MySQL as database management system.

**Cheers!**
