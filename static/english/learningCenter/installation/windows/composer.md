#### Install using composer on windows

This guide explains how to setup application on your machine using composer.

##### Install xampp
XAMPP is the most popular PHP development environment which comes with Apache, PHP & MySQL.

Once downloaded install it in your machine.

We recommend installing it in drive other than _C_. For this tutorial we will consider that you have installed it in _D_ drive.

Download xampp from [here](https://www.apachefriends.org/download.html)

##### Install composer

Download composer from [here](https://getcomposer.org/Composer-Setup.exe) and install it.

##### Install application

1. Open command prompt
2. Go to _D:\xampp\htdocs\_. by typing `cd D:\xampp\htdocs\`
3. Execute below command and wait for installation

```
composer create-project nishchay/nishchay nishchayApp
```

##### Create your site

1. Go to _D:\xampp\apache\conf\extra_
2. Open file _httpd-vhosts.conf_.
3. Add below code at end of file

``` conf
<VirtualHost >
    DocumentRoot "D:\xampp\htdocs\nishchayApp\public"
    ServerName app.nishchay.local
    <Directory "D:\xampp\htdocs\nishchayApp\public">
        Options Indexes FollowSymLinks Includes ExecCGI
        Order allow,deny
        Allow from all
   </Directory>
</VirtualHost>
```
4. lick _next_

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
