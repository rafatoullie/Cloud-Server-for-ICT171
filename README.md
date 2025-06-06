# Overview
I have chosen to make a blog page using wordpress for my project, I will also be including a script that backs up the data for my page everyday.

Following the instructions on our assignment doocument, I will try to write up my documentation in a way that would make it easy to replicate, I am taking inspiration from how our lab work was written.




# Installing Wordpress
Since we have been asked to build this on top of the server we launched for the previous assignment. I will skip documenting the process of starting the web server as this has been assumed to be complete already. The type of instance I am using is a t2.micro instance
from AWS EC2.
Thus, for our first step we must install Wordpress, but before we install Wordpress we need to install a few prerequisites. Namely Apache2 which we already have installed, MySQL to set up a database and PHP since Wordpress is built using it.

- SSH

Firstly we will need to ssh into our AWS EC2 instance using our downloaded key.
We must first change use this code on our key file
```
chmod 0400 key.pem
```
then we must type this in to connect to our instance (assuming we are in the directory with our key file)
```
sudo ssh -i key.pem ubuntu@blog.rafataws.com
```
This should connect us to our instance.

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
then we create a new apache configuration for Wordpress
```
sudo nano /etc/apache2/sites-available/wordpress.conf
```
where I added the following configuration
```
<VirtualHost *:80>
    ServerAdmin admin@yblog.rafataws.com
    DocumentRoot /var/www/html/wordpress
    ServerName blog.rafataws.com
    ServerAlias www.blog.rafataws.com

    <Directory /var/www/html/wordpress/>
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
after which we enable the new site
```
sudo a2ensite wordpress.conf
sudo a2enmod rewrite
```
then after restarting apache with
```
sudo systemctl restart apache2
```
we can complete the wordpress installation from the website itself, for me this code was
```
http://52.62.231.161/wordpress
```


# Building the website

After installing wordpress took me to the dashboard where I was free to build mmy site however I wanted. To build my blog website I used a theme called Blocksy as it was free and had a the kind of template I was looking for, it was called persona.
It wasnt made exclusively for blogging so I had to go to the customiser tab and edit out alot of fluff that i had no need of, including video links and team introductions. I repurposed the website to be as minimal as possible and edited the colours to match the
homepage image I had picked.

I plan on using this website long term, but for now i decided on writing up a few sample posts using ChatGPT. These are only placeholders I am using till I write up an actual post, I am only using AI here to make the site look full of content for now.
After testing the website from both my laptop and my phone, and also making other people check the link to see if anything was wrong with it, The site was complete.

For my next step I wanted to make sure my site was secure as it was still using http/ instead of https/. I looked into the contents of previous lab work from a few weeks prior to get a digital certificate from LetsEncrypt
```
https://certbot.eff.org/
```
As directed from the labwork, I used this website and its instructions, but I will also leave these below.
Firstly, we ssh into our instance and then
```
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache
```
after certification for my domain name was successful I ran a code to renew it automatically post expiry
```
sudo certbot renew --dry-run
```





















