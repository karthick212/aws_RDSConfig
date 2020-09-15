```
sudo su
apt-get update
apt-get upgrade -y
apt-get dist-upgrade -y
apt-get autoremove -y
apt-get install apache2 php7.0 php7.0-cli php7.0-fpm php7.0-gd php-ssh2 libapache2-mod-php7.0 php7.0-mcrypt mysql-server php7.0-mysql git unzip zip postfix php7.0-curl mailutils php7.0-json phpmyadmin -y
php5enmod mcrypt

nano /etc/apache2/sites-enabled/000-default.conf
--ADD LINE-- 
Include /etc/phpmyadmin/apache.conf

service apache2 restart

nano /etc/phpmyadmin/config.inc.php
--ADD LINES BELOW THE PMA CONFIG AREA AND FILL IN DETAILS--
$i++;
$cfg['Servers'][$i]['host']          = '__FILL_IN_DETAILS__';
$cfg['Servers'][$i]['port']          = '3306';
$cfg['Servers'][$i]['socket']        = '';
$cfg['Servers'][$i]['connect_type']  = 'tcp';
$cfg['Servers'][$i]['extension']     = 'mysql';
$cfg['Servers'][$i]['compress']      = FALSE;
$cfg['Servers'][$i]['auth_type']     = 'config';
$cfg['Servers'][$i]['user']          = '__FILL_IN_DETAILS__';
$cfg['Servers'][$i]['password']      = '__FILL_IN_DETAILS__';

//Nodejs Permanently Run
https://stackoverflow.com/questions/12701259/how-to-make-a-node-js-application-run-permanently
```


----SSL EC2 with Apache----
If you are trying to install SSL for one domain in AWS EC2 and its not on AWS ELB. You will have to approach it in different way.

AWS EC2 with Ubuntu installed:

Godaddy ssl cert installation on apache server

To perform a fresh SSL installation, open ssh using puttY and log in to the server. After logging in type ‘sudo su’ so that you can perform all the functions as super admin.

Prepare your server
sudo mkdir /etc/apache2/ssl
chmod 700 /etc/apache2/ssl

chown [file_owner_name]:[name_of_user_group] /etc/apache2/ssl

Install ssl. Nothing happens if already installed
sudo apt-get install openssl
install the required ssl mods for your apache instance and activate them:
sudo a2enmod ssl
Generate CSR public and private key
openssl req -newkey rsa:2048 -nodes -keyout website_name.key -out website_name.csr -sha256
After running the above command you will be required to answer some identity questions. Make sure you answer them as accurate as possible. You don’t really need to use the challenge password with GoDaddy at the time of this writing.
Country Name (2 letter code) [AU]:US

State or Province Name (full name) [Some-State]:New York

Locality Name (eg, city) []:New York City

Organization Name (eg, company) [Internet Widgits Pty Ltd]:Company Name

Organizational Unit Name (eg, section) []:

Common Name (e.g. server FQDN or YOUR name) []:server_IP_address or server_name

Email Address []:admin@your_domain.com

Copy the content of website_name.csr file using vim command in terminal or go to this location in /home/folder/ and then open the .csr file. Copy its content, go to godaddy Rekey & Manage under ssl manage and paste it in the re-key certificate, click save and save all changes.
After receiving email from godaddy, download the certificate files and transfer them to /etc/apache2/ssl
Run the command to move website_name.key to /etc/apache2/ssl
sudo mv ~/website_ssl.key /etc/apache2/ssl/website_name.key
Set the correct permissions
sudo chmod 600 /etc/apache2/ssl/*
sudo chown [file_owner_name]:[name_of_user_group] /etc/apache2/ssl/*
Create apache config file
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.ssl.conf
Copy the following code to 000-default.ssl.conf and do the required changes. Use vim or nano command to edit the above file and the paste code below.
<VirtualHost *:80>
ServerName domain_name.com
ServerAdmin webmaster@localhost
DocumentRoot /var/www/domain_name.com
RewriteEngine On
RewriteCond %{HTTP:X-Forwarded-Proto} !https
RewriteRule ^.*$ https://%{SERVER_NAME}%{REQUEST_URI}
ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
#Include conf-available/serve-cgi-bin.conf
<Directory /var/www/domain_name.com/>
Options Indexes FollowSymLinks
AllowOverride All
Order allow,deny
allow from all
Require all granted
</Directory>
Header always append X-Frame-Options SAMEORIGIN

RequestHeader unset Proxy early

</VirtualHost>

<IfModule mod_ssl.c>

<VirtualHost *:443>

ServerAdmin webmaster@localhost

ServerName domain_name.com

DocumentRoot /var/www/domain_name.com

# SSL Engine Switch:

# Enable/Disable SSL for this virtual host.

SSLEngine on

# A self-signed (snakeoil) certificate can be created by installing

# the ssl-cert package. See

# /usr/share/doc/apache2.2-common/README.Debian.gz for more info.

# if both key and certificate are stored in the same file, only the

# SSLCertificateFile directive is needed.

SSLCertificateFile /etc/apache2/ssl/a06abb0c96113f84.crt

SSLCertificateKeyFile /etc/apache2/ssl/domain_name.key

SSLCertificateChainFile /etc/apache2/ssl/gd_bundle-g2-g1.crt
<Directory /var/www/>
Options Indexes FollowSymLinks
AllowOverride All
Order allow,deny
allow from all
Require all granted
</Directory>
</VirtualHost>
</IfModule>

Enable the secure configuration by using the a2ensite command
sudo a2ensite 000-default.ssl.conf
Restart apache server
Sudo service apache2 restart
To check whether the ssl installed successfully login to godaddy account click on manage ssl, click on ssl checker and type the domain
https://ssltools.godaddy.com/views/certChecker
If you are installing it on Linux AMI 2015 then follow the link below:
How to install an SSL certificate on Amazon EC2 Linux AMI

You might get error while following the method in the above link
“

Error: httpd24-tools conflicts with httpd-tools-2.2.34-1.16.amzn1.x86_64

Error: httpd24 conflicts with httpd-2.2.34-1.16.amzn1.x86_64

You could try using --skip-broken to work around the problem

You could try running: rpm -Va --nofiles --nodigest

”

instead of executing command sudo yum install -y mod_ssl
replace mod_ssl with mod24_ssl and then run the command
