# Overview
Name: Rafat Rahman  
Student Number: 35673718

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
CREATE DATABASE wpdb DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wpdb.* TO 'wpuser'@'localhost';
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

I plan on using this website long term, but for now I decided on writing up a few sample posts using ChatGPT. These are only placeholders I am using till I write up an actual post, I am only using AI here to make the site look full of content for now.
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


# EDIT

After following the steps to obtain a digital certificate, my website no longer worked. It did not load in any platform and I had no way to access it, not even through my wordpress admin panel.
I spent an entire day trying to troubleshoot this issue but unfortunately nothing worked. After this incident I had no other option except redoing the entire process. Fortunately, this documentation helped
streamline the process. The only thing that I did differently was acquiring a digital certificate right after installing apache for the  new instance. I used the same domain name with the same hosted zone,
I only edited out the previous ip address with the updated one. The new ip is
```
3.27.167.100
```

While I was redoing my website I noticed that my wordpress website could only be accessed via the ip address and not the domain name. This was caused by the Apache virtual host config pointing to the root folder 
in var/www/html, but my wordpress files were located in a sub-folder inside that root folder. To fix this, I moved the contents into the root folder and edited my general settings in wordpress to use my domain name
instead.

```
sudo mv /var/www/html/wordpress/* /var/www/html/
sudo mv /var/www/html/wordpress/.* /var/www/html/ 2>/dev/null
sudo rm -rf /var/www/html/wordpress
sudo rm /var/www/html/index.html
sudo systemctl reload apache2
```

# SCRIPT

For the scripting component, I decided to write a backup script that will run everyday to ensure my data is safe incase any potential issue arises. To do this we need to first create a directory to store our backups
```
sudo mkdir -p /home/ubuntu/backups
sudo chown ubuntu:ubuntu /home/ubuntu/backups
```
after which we can create the script
```
nano /home/ubuntu/backup_wordpress.sh
```
then we have to type in the script.
```
#!/bin/bash

DATE=$(date +'%Y-%m-%d_%H-%M-%S')
BACKUP_DIR="/home/ubuntu/backups"
WEB_DIR="/var/www/html"
DB_NAME="wpdb"
DB_USER="wpuser"
DB_PASS="(my password)"

mysqldump -u $DB_USER -p$DB_PASS $DB_NAME > $BACKUP_DIR/dbbackup$DATE.sql
tar -czf $BACKUP_DIR/sitebackup$DATE.tar.gz $WEB_DIR

find $BACKUP_DIR -type f -mtime +7 -name ".tar.gz" -exec rm {} ;
find $BACKUP_DIR -type f -mtime +7 -name ".sql" -exec rm {} ;

echo "Backup completed on $DATE"
```
I have left out my password in the script I have provided above.
After this we have to make this script executable using
```
chmod +x /home/ubuntu/backup_wordpress.sh
```
Then, after checking if the script works properly, we can use the following lines to automate this.
```
crontab -e
```
Here we paste the following script which will run this backup script every night at 3:00 AM and all output will be appended to the backup log file.
```
0 3 * * * /home/ubuntu/backup_wordpress.sh >> /home/ubuntu/backup_log.txt 2>&1
```


# Conclusion

This assignment took me longer than expected as I made the mistake of not securing my server beforehand, however this turned out to be a net positive as it resulted in me redoing the entire process by following the
instructions I wrote in this document. It allowed me to iron out mistakes and update a few lines. 

My blog site is not fully functional with an automated backup script. I created it using Wordpress and I got my domain name from Route53.  
domain: blog.rafataws.com

















