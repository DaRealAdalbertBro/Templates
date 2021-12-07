# HOW TO SETUP A SUB-DOMAIN WITH APACHE2 ON Ubuntu 21.04
> Pro tip: just replace subdomain and mydomain variable with your sub-domain and domain, you will save a lot of time with rewriting



### 1.) First of all, you need to go to your domain provider's site and add the sub-domain on the list (create an A record - subdomain.mydomain.com - that directs to your web server).

### 2.) Create a directory for your sub-domain on your server:
```
mkdir -p /var/www/subdomain.mydomain.com/html && touch /var/www/subdomain.mydomain.com/html/index.html
```

## 3.) Create a config file:
```
sudo nano /etc/apache2/sites-available/subdomain.mydomain.com.conf
```

## 4.) Paste the following code into your config file:

```
<VirtualHost *:80>
    ServerName subdomain.mydomain.com
    ServerAlias www.subdomain.mydomain.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/subdomain.mydomain.com/html

    <Directory /var/www/subdomain.mydomain.com/html/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/subdomain.mydomain.com-error.log
    CustomLog ${APACHE_LOG_DIR}/subdomain.mydomain.com-access.log combined
    
    RewriteEngine on
    RewriteCond %{SERVER_NAME} =subdomain.mydomain.com [OR]
    RewriteCond %{SERVER_NAME} =www.subdomain.mydomain.com
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

## 5.) Save the file with CTRL+X, then press Y and ENTER. Now you can enable the subdomain and reload apache:
```
sudo a2ensite subdomain.mydomain.com
sudo systemctl restart apache2
```

## 6.) Almost done! You just need to edit the following file and add your sub-domain to it:
```
sudo nano /etc/hosts
```

##### And add your sub-domain after your main domain, so there will be something like this:
```
127.0.1.1 mydomain mydomain subdomain.mydomain anotherSubDomain.mydomain
127.0.0.1 localhost

. . .
```

##### The following "mydomain" will be without suffix (.com, .eu, etc.)

# How to generate a certificate

##### 1.) If you are using a certbot, its easy, just put this command to a command line:
```
certbot -d subdomain.mydomain.com --expand
```
##### or for more domains:
```
certbot -d subdomain.mydomain.com,anotherSubDomain.mydomain.com --expand
```
