# How to setup a sub-domain with Apache2 on Ubuntu 21.04
> Pro tip: just replace subdomain and mydomain variable with your sub-domain and domain, you will save a lot of time with rewriting

<br /><br /><br />

## Step 1 - Create an A record in your DNS settings
##### First of all, you need to go to your domain provider's site and add the sub-domain on the list (create an A record - subdomain.mydomain.com - that directs to your web server).<br /><br />

## Step 2 - Create a directory for your sub-domain
```
mkdir -p /var/www/subdomain.mydomain.com/html && touch /var/www/subdomain.mydomain.com/html/index.html
```
<br />

## Step 3 - Create an Apache2 config file
```
sudo nano /etc/apache2/sites-available/subdomain.mydomain.com.conf
```
<br />

## Step 4 - Setup sub-domain config file
```
<VirtualHost *:80>
    ServerName subdomain.mydomain.com
    ServerAlias www.subdomain.mydomain.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/subdomain.mydomain.com/html

    <Directory /var/www/subdomain.mydomain.com/html/>
        Options -Indexes +FollowSymLinks
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
##### Save the file with CTRL+X, then press Y and ENTER.
<br /><br />
## Step 5 - Enable the subdomain and reload apache
```
sudo a2ensite subdomain.mydomain.com
sudo systemctl restart apache2
```

## Step 6 - Edit "hosts" file

##### Almost done! You just need to edit the following file and add your sub-domain to it:

```
sudo nano /etc/hosts
```

##### And add your sub-domain after your main domain, so there will be something like this:

```
127.0.1.1 mydomain mydomain subdomain.mydomain anotherSubDomain.mydomain
127.0.0.1 localhost

. . .
```
<br />

##### The following "mydomain" will be without suffix (.com, .eu, etc.)

<br /><br /><br />

# How to generate a certificate

## Step 1 - Installing certbot

```
sudo apt install certbot python3-certbot-apache
```

<br />

## Step 2 - Obtaining an SSL Certificate

```
sudo certbot --apache
```

<br />

##### Enter a recovery email address, press A [ENTER], press N [ENTER]. Choose Redirect (=2) [ENTER]

## Situational - Expand certificate

```
certbot -d subdomain.mydomain.com --expand
```

<br />

##### Or for more domains:

```
certbot -d subdomain.mydomain.com,anotherSubDomain.mydomain.com --expand
```

# Creating .htaccess file

##### File base:

```
Options +FollowSymLinks -MultiViews
# Turn mod_rewrite on
RewriteEngine On
RewriteBase /
RewriteCond %{HTTP_HOST} !=localhost
RewriteCond %{HTTP_HOST} !=127.0.0.1
RewriteCond %{REMOTE_ADDR} !=127.0.0.1
RewriteCond %{REMOTE_ADDR} !=::1
```

<br />

##### To externally redirect /dir/foo.html to /dir/foo

```
RewriteCond %{REQUEST_URI} !(\.[^./]+)$
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME}.html -f
RewriteRule (.*) $1.html [L]
```

<br />

##### To externally redirect /dir/foo.htm to /dir/foo

```
RewriteCond %{REQUEST_URI} !(\.[^./]+)$
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME}.htm -f
RewriteRule (.*) $1.htm [L]
```

<br />

##### To externally redirect /dir/foo.php to /dir/foo

```
RewriteCond %{THE_REQUEST} ^[A-Z]{3,}\s([^.]+)\.php [NC]
RewriteRule ^ %1 [R=302,L]

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME}.php -f
RewriteRule ^(.*?)/?$ $1.php [L]
```

<br />

##### End of the file (depends on permissions):

```
<FilesMatch "^\.">
    Order allow,deny
    Deny from all
</FilesMatch>
```
