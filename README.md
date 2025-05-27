# Overview
I have chosen to make a blog page using wordpress for my project, I will also be including a script that backs up the data for my page everyday.




# Installing Wordpress
Since we have been asked to build this on top of the server we launched for the previous assignment. I will skip documenting the process of starting the web server as this has been assumed to be complete already. The type of instance I am using is a t2.micro instance
from AWS EC2.
Thus, for our first step we must install Wordpress, but before we install Wordpress we need to install a few prerequisites. Namely MySQL to set up a database and PHP since Wordpress is built using it.

- MySQL  

We can install MySQL by using the following code
```
sudo apt install mysql-server
```
Then we have to follow a few prompts to secure the MySQL server, such as setting a root password and removing anonymous users.

```
sudo mysql_secure_installation
```
Then we can log into mySQL with the following code

```
sudo mysql -u root -p
```
Then we need to create a new user and database for Wordpress. We do this by typing in the following code sequentially

```
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

```
replace 'password' with any password of choice.


- PHP

To install PHP and all required extensions we use this line of code

```
sudo apt install php libapache2-mod-php php-mysql php-xml php-mbstring php-curl php-zip
```
and then we need to restart apache to load PHP

```
sudo systemctl restart apache2
```

- Wordpress

Now that we have installed all prerequisites we can install Wordpress, we can use wget to install it

```
wget https://wordpress.org/latest.tar.gz
```

then we can extract and move the files to the web root
```
tar -xvf latest.tar.gz
```
```
sudo mv wordpress /var/www/html/wordpress
```
then set permissions
```
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
```























