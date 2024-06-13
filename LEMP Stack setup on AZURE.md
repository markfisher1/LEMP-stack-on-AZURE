# Setting Up LEMP Stack on AZURE:

# LEMP stands for: Linux, Nginx (engine x) web server, MySQL Database, and PHP


# 1.Connect to a Virtual Machine:


# Step 1: Create a Virtual Machine

The first thing we are going to do is to log into in the AZURE Console and create a VM
Choose a name you like and select the Ubuntu image for the VM.
Select the “Ubuntu Server 22.04 LTS HVM, Volume Type”.
You will see it under the free tier.
Select the default EBS volume(8 gbs)

Under the “Network settings” we will create a new security group and select “allow SSH 80 traffic from” and “Allow HTTP 443 from the internet”.
This will allow our instance to be able to send and receive traffic, thus be able to connect to the internet.


# Step 2: via CLI

# i. ON YOUR COMMAND PROMPT Connect to Instance:
Establish SSH connection to the VM --

cd "path to your key file" for eg- cd C:\Users\NAME\linux key  (remember to put the cd command before the path)
then, click enter and copy and paste your AZURE CONSOLE SSH (its found when you click on your VM name and click on connect, 
under the 'connect to VM' page click on the SSH client and you will find your VM public DNS that
looks like this az ssh vm --resource-group krystal_group --vm-name krystal --subscription b7e6d027-5a13-4ec9-ae33-10e9a7e0ce13
                   
                                OR

# ii. TO CONNECT to your VM VIA YOUR AZURE CLI [BASH OR POWERSHELL (UBUNTU)]-  
ON YOUR AZURE CONSOLE click on your VM name and cick on 'connect',
It will validate, configure and then click on connect.

update to the latest version by running the command - 

sudo apt update

if there is anything to ugrade - 

sudo apt upgrade


# 2.Install Components:

Install Nginx, MySQL, and PHP packages using package manager commands--
 

# a. Installing Nginx --

sudo apt install nginx

check status of the NGINX web server:--

sudo systemctl status nginx
sudo systemctl enable nginx

Check to see if we can receive traffic to our VM via the CLI.
If everything was setup properly you should see the default NGINX webpage

Run the following command:--
curl http://localhost:80

Check to see if we can receive traffic to our VM via the WEB BROWSER.
Copy the IP address of the VM and paste that into your browser window using the ,
and hit the ENTER/RETURN key on your keyboard.--

http://<Public-IP-Address>:80

 voila 


# b. Installing MySQL

Our web server is up and running so now it is time to focus on the “M” part of the LEMP stack. The “M” stands for MySQL.
MySQL is a popular relational database management system used for PHP environments,
and we will deploying that as part of this project!

Run the following command to install MySQL:

When prompted enter “Y” and hit enter/return key--

sudo apt install mysql-server

After the successful installation of the MySQL database, we are going to log into the MySQL console.

Run the command:
sudo mysql

We will run a security script directly from the MySQL console, and this script will give us the permission 
to set a password for our example root user to be able to have access to the database.

Run the following script from the MySQL console: “mysql>” --:

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

Exit the MySQL console --

mysql> exit

Next, we are going to run a few scripts to validate the password configuration

Run the command, enter “Y” when prompted--

sudo mysql_secure_installation

When prompted with the validation policy, enter the value 1. The value 1 represents the medium value that is being applied to the root password.
You will be met with a few confirmation prompts, enter “Y” for all.

Now that our root user password has been SET. We will reconnect to MySQL to save the changes and exit.
MySQL is now officially installed and secured.


# c. Installing PHP

So far our we have successfully deployed a linux VM, NGINX web server, and MySQL relational database. 
It is time to deploy the last part of the LEMP stack, and that is PHP.
For this project we will setup the PHP to be able to communicate with the NGINX web server so that it can host our custom website.

PHP- “A popular general-purpose scripting language that is especially suited to web development.
Fast, flexible and pragmatic, PHP powers everything from your blog to the most popular websites in the world”.- php.net

Run the following command to install PHP:

These two command will install php, and allow it to work with NGINX.
Enter “Y” when prompted and hit the ENTER/RETURN key on your keyboard.--

sudo apt install php-fpm php-mysql


# 3. Configuring NGINX to work with PHP

By default NGINX is setup to only run one server block.
Next, you'll need to configure Nginx to process PHP files.
By default, Nginx doesn't handle PHP files directly.
You'll need to configure it to pass PHP requests to the PHP FastCGI Process Manager (PHP-FPM).
We are going to change the default settings so that we can enable the server block to allow for multiple server configurations.
We will create a new directory to host our PHP based website.

Run the following command to create the root directory: We will name the root directory “/var/www/projectLEMP” --

sudo mkdir /var/www/projectLEMP

We will change ownership of the directory and assign it to our root user($USER) --

sudo chown -R $USER:$USER /var/www/projectLEMP

Using the CLI editor, open a new configuration file in Nginx’s sites-available directory. Use nano.--

sudo nano /etc/nginx/sites-available/projectLEMP

Copy and paste the script below and save and exit the file.--

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


“listen”= Tells NGINX what port to listen on. Port 80 is the port for HTTP

“root”= Where the root document will be stored

“index”= refers to the index.html, index.htm, as well as the index.php files for the application.


The following script will activate the configuration file from the NGINX’s sites-enabled directory:--

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

We want NGINX web server to serve the configuration file as opposed to the default NGINX webpage when the page is reloaded.
The following script will check the file for any errors..--

sudo nginx -t

If there are any syntax errors make sure to go back and check the configuration.

We must disable the default NGINX host that is listening on port 80.
Afterwards, we will reload NGINX to save the changes.

Run the following command to disable it.--

sudo unlink /etc/nginx/sites-enabled/default
( removes nginx as the default host )

sudo systemctl reload nginx
( reloads nginx )


# 4.Test LEMP stack - Create and access a PHP info file to confirm successful stack setup.

Our custom website is now up and running, but we have to create an index.html 
so that we can test out the server block and make sure things are working properly.--

sudo sh -c "echo 'Hello LEMP from hostname $(curl -s http://your public IP address/latest/meta-data/public-hostname) with public IP $(curl -s http://your public IP adress/latest/meta-data/public-ipv4)' > /var/www/projectLEMP/index.html"

for example
sudo sh -c "echo 'Hello LEMP from hostname $(curl -s http://20.190.196.102/latest/meta-data/public-hostname) with public IP $(curl -s http://20.190.196.102/latest/meta-data/public-ipv4)' > /var/www/projectLEMP/index.html"

The last leg of the LEMP stack is now complete! In next step we will create a PHP script to test with NGINX!


# Testing PHP with NGINX

We are going to test and validate that NGINX can handle PHP files and serve them on our website.

Before we can run a test, a test PHP file will need to be created in our root directory using the text editor.
Run the following command.--

sudo nano /var/www/projectLEMP/index.php

Paste the following text in the new file.
This PHP code will give us information about our server.
Save and exit the file.--

<?php
phpinfo();
?>


With the PHP file setup we can now access the webpage in the web browser.
You will use the public IP of the instance or the domain name you setup in the NGINX configuration file,
followed by/index.php.

In your browser type the following and hit ENTER/RETURN key.--

http://`server_domain_or_IP`/index.php

for example -
http://44.203.190.129/index.php

You should see the webpage, Detailed information about your server will be shown. Scroll down and take a look!

Great! Good job the PHP file was able to be served up by the PHP server. Everything looks good! Now that you have taken a good look at the server information, it is time to remove the file. The PHP contains sensitive information about your PHP environment and Ubuntu server(instance).

Run the following command to remove the PHP file:
sudo rm /var/www/your_domain/index.php



# NOTES

sudo apt install -y software-properties-common

sudo add-apt-repository ppa:ondrej/php

sudo apt update

sudo apt install nginx 

sudo apt install mysql-server

sudo apt install php-fpm php-mysql

sudo mkdir /var/www/projectLEMP

sudo chown -R $USER:$USER /var/www/projectLEMP

sudo nano /etc/nginx/sites-available/projectLEMP

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

sudo nginx -t

sudo unlink /etc/nginx/sites-enabled/default

sudo systemctl reload nginx

sudo sh -c "echo 'Hello LEMP from hostname $(curl -s http://your public IP address/latest/meta-data/public-hostname) with public IP $(curl -s http://your public IP adress/latest/meta-data/public-ipv4)' > /var/www/projectLEMP/index.html"

http://`server_domain_or_IP`/index.php
