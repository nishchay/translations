#### Install without composer on windows

This guide explains how to setup application on your machine without composer.

##### Download

You first need to download application, if you already have downloaded application skip this step.

1. Go to home page
2. Download application

##### Install xampp
XAMPP is the most popular PHP development environment which comes with Apache, PHP & MySQL.

Once downloaded install it in your machine.

We recommend installing it in drive other than _C_. For this tutorial we will consider that you have installed it in _D_ drive.

Download xampp from [here](https://www.apachefriends.org/download.html)

##### Copy application

1. Extract downloaded application
2. Copy and past it in _D:\xampp\htdocs\_.

##### Create your site

1. Go to _D:\xampp\apache\conf\extra_
2. Open file _httpd-vhosts.conf_.
3. Add below code at end of file

``` conf
<VirtualHost >
    DocumentRoot "D:\xampp\htdocs\nishchay\public"
    ServerName app.nishchay.local
    <Directory "D:\xampp\htdocs\nishchay\public">
        Options Indexes FollowSymLinks Includes ExecCGI
        Order allow,deny
        Allow from all
   </Directory>
</VirtualHost>
```

##### Add host entry

1. Open <i>C:\Windows\system32\drivers\etc\hosts</i> with administritive access so that you edit and save changes.
2. Add below line at end of file.

``` 
127.0.0.1   app.nishchay.local
```

##### Restart apache server</h3>

1. Open xampp control panel(It might at right bottom of your screen in toolbar section).
2. Stop and restart apache server.

##### Check if it's working now

1. Open browser
2. Visit [http://app.nishchay.local](http://app.nishchay.local)
3. You should see application welcome page.
