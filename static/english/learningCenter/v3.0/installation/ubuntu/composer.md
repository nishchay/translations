#### Install on Ubuntu using Composer

This guide installs Nishchay on Ubuntu with **Composer**, **NGINX** and **PHP-FPM**. It was written for Ubuntu 24.04, and the same steps apply to 22.04. It takes about ten minutes.

##### Before you start

- **PHP 8.3 or newer.** Older versions have reached end of life and are not supported. See the [supported PHP versions](https://www.php.net/supported-versions.php). The commands in this guide install PHP 8.4. Ubuntu's own repositories can ship an older PHP (22.04 has 8.1), so this guide adds a trusted PHP repository that provides current versions.
- **Composer**, to download the framework and its dependencies.
- **A user with `sudo` access.**

##### Install PHP and NGINX

Update the package index, then add the PHP repository maintained by Ondřej Surý, which provides every PHP version for Ubuntu.

```bash
sudo apt update
sudo apt install -y software-properties-common ca-certificates curl unzip git
sudo add-apt-repository -y ppa:ondrej/php
sudo apt update
```

Install NGINX, PHP 8.4 and the extensions the framework uses.

```bash
sudo apt install -y nginx php8.4-fpm php8.4-cli php8.4-mbstring php8.4-xml php8.4-curl php8.4-mysql php8.4-gd php8.4-zip
```

Check the version. It must say `PHP 8.3` or newer.

```bash
php -v
```

If another PHP is installed and `php -v` shows a different version, select 8.4:

```bash
sudo update-alternatives --set php /usr/bin/php8.4
```

Start both services and make them start on boot.

```bash
sudo systemctl enable --now php8.4-fpm nginx
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

Create `/var/www` if it does not exist, and make it yours so you do not need `sudo` or `chmod 777` to work in it.

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

Open [http://localhost:8000](http://localhost:8000) and you should see the Nishchay welcome page. Press `Ctrl+C` to stop it. The built-in server is for trying things out, so continue below to set up NGINX for real use.

##### Give the web server write access

The application writes logs and caches views and sessions. PHP-FPM runs as `www-data`, so let it write to those two folders.

```bash
sudo chown -R www-data:www-data /var/www/nishchayApp/logs /var/www/nishchayApp/persisted
```

##### Create your site

Create the NGINX configuration.

```bash
sudo nano /etc/nginx/conf.d/nishchay-app.conf
```

Paste the following, then save with `Ctrl+O`, `Enter`, `Ctrl+X`.

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
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.4-fpm.sock;
    }

    # Never serve hidden files such as .git.
    location ~ /\. {
        deny all;
    }
}
```

The root points at the application's **`public`** folder. Never point it at the project folder itself, because that would expose your settings.

Check the configuration, then reload NGINX.

```bash
sudo nginx -t
sudo systemctl reload nginx
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
| **502 Bad Gateway** | PHP-FPM is not running or the socket path differs. Run `sudo systemctl status php8.4-fpm` and check that `fastcgi_pass` matches the `listen` line in `/etc/php/8.4/fpm/pool.d/www.conf`. |
| **404 on every page except `/`** | The `try_files $uri $uri/ /index.php?$args;` line is missing from `location /`. |
| **An error about your platform, or a PHP parse error** | The PHP version is too old. Run `php -v`, see _Install PHP and NGINX_ above, and check which version PHP-FPM uses in `fastcgi_pass`. |
| **Blank page, or a permission error in the NGINX log** | The `logs` and `persisted` folders are not writable by `www-data`. Repeat the `chown` step above. |
| **Failed opening required `./../bootstrap/init.php`** | You started the built-in server from the wrong folder. Start it from inside `public`. |
| **Cannot reach the site from another machine** | Allow HTTP through the firewall: `sudo ufw allow 'Nginx HTTP'`. |
| **NGINX will not reload** | Run `sudo nginx -t`. It prints the file and line with the mistake. |

##### Next

- [Create your first controller and route](/learningCenter/getStarted/web)
- [Build a REST API](/learningCenter/getStarted/restApi)
- [Understand how a request is handled](/learningCenter/lifeCycle)
