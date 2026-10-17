#### Install on CentOS, Rocky Linux or AlmaLinux using Composer

This guide installs Nishchay with **Composer**, **NGINX** and **PHP-FPM** on the Red Hat family of Linux: CentOS Stream, Rocky Linux and AlmaLinux, version 8 or 9. It was written for AlmaLinux 9. The classic CentOS 7 and 8 releases have reached end of life, so use one of the maintained systems above.

##### Before you start

- **PHP 8.3 or newer.** Older versions have reached end of life and are not supported. See the [supported PHP versions](https://www.php.net/supported-versions.php). The commands in this guide install PHP 8.4 from the Remi repository, which provides current PHP versions for this family.
- **Composer**, to download the framework and its dependencies.
- **A user with `sudo` access.**

##### Install PHP and NGINX

Add the EPEL and Remi repositories.

```bash
sudo dnf install -y epel-release
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-$(rpm -E %rhel).rpm
```

Switch the PHP module to the Remi 8.4 stream.

```bash
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.4 -y
```

Install NGINX, PHP 8.4 and the extensions the framework uses.

```bash
sudo dnf install -y nginx php-fpm php-cli php-mbstring php-xml php-mysqlnd php-gd php-zip unzip git
```

Check the version. It must say `PHP 8.3` or newer.

```bash
php -v
```

##### Install Composer

Download the installer, check its signature, then install it.

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
HASH="$(curl -sS https://composer.github.io/installer.sig)"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
rm composer-setup.php
```

You should see `Installer verified`. Confirm that Composer works:

```bash
composer --version
```

##### Install the application

Create `/var/www` and make it yours, so you do not need `sudo` to work in it.

```bash
sudo mkdir -p /var/www
sudo chown $USER:$USER /var/www
cd /var/www
```

Create the project.

```bash
composer create-project nishchay/nishchay nishchayApp
```

The installer asks for **your application name** and **your or your organisation's name**, then generates an encryption key and a service app id and secret. They are saved in `settings/configuration/`, and you can change them later.


##### Try it right away

You do not need a web server to see the application. PHP has a built-in one. Start it from inside the `public` folder.

```bash
cd /var/www/nishchayApp/public
php -S localhost:8000
```

Open [http://localhost:8000](http://localhost:8000) in a browser on the same machine and you should see the Nishchay welcome page. Press `Ctrl+C` to stop it. The built-in server is for trying things out, so continue below to set up NGINX for real use.

##### Run PHP-FPM as the NGINX user

On this family of systems PHP-FPM runs as `apache` by default. Change it to `nginx`, so that NGINX and PHP agree on file ownership.

```bash
sudo sed -i -e 's/^user = apache/user = nginx/' -e 's/^group = apache/group = nginx/' /etc/php-fpm.d/www.conf
```

Then give that user write access to the two folders the application writes to, for logs, cached views and sessions.

```bash
sudo chown -R nginx:nginx /var/www/nishchayApp/logs /var/www/nishchayApp/persisted
```

##### Create your site

Create the NGINX configuration.

```bash
sudo vi /etc/nginx/conf.d/nishchay-app.conf
```

Paste the following. In `vi`, press `i` to type, then `Esc` and `:wq` and `Enter` to save.

```nginx
server {
    listen 80;
    server_name app.nishchay.local;
    root /var/www/nishchayApp/public;
    index index.php;

    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    charset utf-8;

    # Everything that is not a real file goes to the application.
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/run/php-fpm/www.sock;
    }

    # Never serve hidden files such as .git.
    location ~ /\. {
        deny all;
    }
}
```

The root points at the application's **`public`** folder. Never point it at the project folder itself, because that would expose your settings.

Start both services, make them start on boot, and check the configuration.

```bash
sudo nginx -t
sudo systemctl enable --now php-fpm nginx
```

##### Open the firewall

If `firewalld` is running and you want to reach the site from another machine:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

##### If SELinux is enforcing

Run `getenforce`. If it prints `Enforcing`, SELinux must be told that NGINX and PHP may read the application and write to its two folders. Without this you get a blank page or a 403 even though the file permissions are correct.

```bash
sudo dnf install -y policycoreutils-python-utils
sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/nishchayApp(/.*)?"
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/nishchayApp/(logs|persisted)(/.*)?"
sudo restorecon -Rv /var/www/nishchayApp
```

If your application connects to a database, also allow that:

```bash
sudo setsebool -P httpd_can_network_connect_db 1
```

##### Add a host entry

This makes `app.nishchay.local` open the site on your own machine.

```bash
echo "127.0.0.1   app.nishchay.local" | sudo tee -a /etc/hosts
```

On a real server, point a DNS record at the server and use that domain as `server_name` instead.

##### Check that it works

Open [http://app.nishchay.local](http://app.nishchay.local). You should see the Nishchay welcome page.

##### If something is wrong

| What you see | What to do |
| ------------ | ---------- |
| **502 Bad Gateway** | PHP-FPM is not running, or NGINX cannot open its socket. Run `sudo systemctl status php-fpm`, and check that `fastcgi_pass` matches the `listen` line in `/etc/php-fpm.d/www.conf`. |
| **403 Forbidden or a blank page**, with correct permissions | SELinux. See _If SELinux is enforcing_ above, and look at `sudo ausearch -m avc -ts recent`. |
| **404 on every page except `/`** | The `try_files $uri $uri/ /index.php?$args;` line is missing from `location /`. |
| **An error about your platform, or a PHP parse error** | The PHP version is too old. Run `php -v` and see _Install PHP and NGINX_ above. |
| **Failed opening required `./../bootstrap/init.php`** | You started the built-in server from the wrong folder. Start it from inside `public`. |
| **Cannot reach the site from another machine** | Open the firewall, as shown above. |
| **NGINX will not start** | Run `sudo nginx -t`. It prints the file and line with the mistake. |

##### Next

- [Create your first controller and route](/learningCenter/getStarted/web)
- [Build a REST API](/learningCenter/getStarted/restApi)
- [Understand how a request is handled](/learningCenter/lifeCycle)
