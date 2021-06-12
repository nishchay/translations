#### Install using composer on CentOS

This guide will explain how to setup application on CentOS.

##### Prerequisite

In order to install PHP and server we first need to install EPEL repository which contains additional softwares for the centOS.

```
sudo yum install epel-release
```

##### Install NGINX

Execute below command to install NGINX. NGINX is server which will serve our application from browser or any other tool.

```
sudo yum install nginx
```

Check nginx has been installed and its running.

```
sudo systemctl start nginx
```

##### Install PHP

To install PHP we first need to install REMI repository which contains all PHP versions.

Install REMI repository by executing below command:

```
sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

Now enable repo

```
yum --disablerepo="*" --enablerepo="remi-safe"
sudo yum-config-manager --enable remi-php74
```

Install PHP

```
sudo yum install php php-fpm php-cli php-curl php-mysql php-curl php-gd php-mbstring php-pear
```

##### Install composer
Download installer:

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
```

Verify installer. To verify installer we will first fetch latest signature and store it in variable.

```
HASH="$(wget -q -O - https://composer.github.io/installer.sig)"
```

Now run below command to check if the installer is not corrupted.

```
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

Install

```
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

Confirm composer has been installed.

```
composer
```

##### Install application

1. Go to `/var/www` directory. If `www` does not exists then create it using below command

```cmd
cd /var
sudo mkdir www
sudo chmod 777 www
```

2. Now we will install application in above directory.

```
composer create-project nishchay/nishchay nishchayApp
```

Wait for installation to complete. It will ask for your name and application, enter details when it prompts.

##### Create site

Let's create site for our application.

1. Go to _/etc/nginx/conf.d_ by executing `cd /etc/nginx/conf.d`.
2. Execute `sudo touch nishchay-app.conf`.
3. Execute `sudo vim nishchay-app.conf`.

This will open editor in terminal window, Paste below code:

```
server {
    listen 80;
    server_name app.nishchay.local;
    root   '/var/www/nishchayApp/public';
    index index.php;

    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options nosniff;

    charset utf-8;
    add_header 'Access-Control-Allow-Origin' '*' always;

    location / {
        if ($request_method = 'POST|GET|PUT|DELETE') {
                add_header 'Access-Control-Allow-Origin' '*' always;
        }

        index  index.html index.php;
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    # prevent nginx from serving dotfiles (.htaccess, .svn, .git, etc.)
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

}
```

If installed PHP version is different then edit this line in above configuration as per installed php version:

```
fastcgi_pass unix:/run/php/php7.4-fpm.sock;
```

To save file

1. Press `escape`.
2. Type `:wq` and press enter.


##### Check site configuration is correct

Execute below command which will check if nginx files are correct or not.

```
sudo nginx -t
```

If You receive any error, please correct it and save it then check again by executing above command.


###### Restart NGINX

To restart NGINX server execute below command:

```
systemctl restart nginx
```


##### Add host entry

Execute below command to open editor in terminal

```
sudo vim /etc/hosts
```

Add below line at the end

```
127.0.0.1   app.nishchay.local
```

##### Check if it's working now

1. Open browser.
2. Visit [http://app.nishchay.local](http://app.nishchay.local)
3. You should see application welcome page.