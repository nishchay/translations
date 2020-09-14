#### Install without composer on Mac

This guide explains how to setup application on your machine without composer.

##### Download

You first need to download application, if you already have downloaded application skip this step.

1. Go to home page
2. Download application


##### Install brew

Visit [Brew](https://docs.brew.sh/Installation) and install brew. This will help us install all the tool required to run our application.

##### Install NGINX

To install nginx execute below command. NGINX is server which will serve our application.

```
brew install nginx
```

Configure it to auto start when machine restarts.

```
sudo cp -v /usr/local/opt/nginx/homebrew.mxcl.nginx.plist /Library/LaunchDaemons/
sudo chown root:wheel /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```

Change default port.

```
sudo vim vim /usr/local/etc/nginx/nginx.conf
```

Change `listen 8080` to `listen 80`. If its already 80, then keep as it is.

Save file by pressing `escape` and type `:wq` then press enter.

Start NGINX

```
sudo nginx
```

If it gives error then try below command

```
brew services start nginx
```

##### Install PHP

Execute below command one by one

```
brew tap homebrew/dupes
brew tap homebrew/ph
rew install --without-apache --with-fpm php73
```

If it gives error try to changing `php73` with `php@7.3`, `php@73`. Once of it will surely work.

Start php
```
sudo brew services start php73
```

##### Install composer

Execute below command

```
brew install composer
```

##### Copy downloaded application to server location

1. Go to directory where you downloaded application
2. Extract zip file
3. Then execute below command:

```
cp {DOWNLOAD PATH}/nishchay /var/www/nishchay
```

Now check following path is correct:

```
/var/www/nishchay/public
```


##### Create your site

Let's create site for our application.

1. Go to _/usr/local/etc/nginx/conf.d_ by executing `cd /usr/local/etc/nginx/conf.d`.
2. Execute `sudo touch nishchay-app.conf`.
3. Execute `sudo vim nishchay-app.conf`.

This will open editor in terminal window, Paste below code:

```
server {
    listen 80;
    server_name app.nishchay.local;
    root   '/user/local/var/www/nishchay/public/';
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
        try_files $uri =404;
        include        fastcgi.conf;
        fastcgi_pass   127.0.0.1:9000;
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

##### Check site configuration is correct

Execute below command which will check if nginx files are correct or not.

```
sudo nginx -t
```

If You receive any error, please correct it and save it then check again by executing above command.

###### Restart NGINX

To restart NGINX server execute below command:

```
brew services restart nginx
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