#### Install on macOS using Composer

This guide installs Nishchay on macOS with **Homebrew**, **Composer**, **NGINX** and **PHP-FPM**. It works on both Apple Silicon and Intel Macs. If you only want to look at the framework first, the _Try it right away_ step needs no web server at all.

##### Before you start

- **PHP 8.3 or newer.** Older versions have reached end of life and are not supported. See the [supported PHP versions](https://www.php.net/supported-versions.php). The commands in this guide install PHP 8.4, and Homebrew's plain `php` formula (the newest version) works too.
- **Homebrew.** Install it from [brew.sh](https://brew.sh) if you do not have it.
- **A terminal** (Terminal or iTerm). `$(brew --prefix)` in the commands below prints `/opt/homebrew` on Apple Silicon and `/usr/local` on Intel, so you can paste them as they are.

##### Install PHP, NGINX and Composer

Install PHP 8.4 and NGINX.

```bash
brew install php@8.4 nginx
```

A versioned PHP is installed alongside other versions and is not put on your path automatically. Add it to the front of your path so that `php` means 8.4.

```bash
echo 'export PATH="$(brew --prefix)/opt/php@8.4/bin:$(brew --prefix)/opt/php@8.4/sbin:$PATH"' >> ~/.zshrc
source ~/.zshrc
php -v
```

`php -v` must say `PHP 8.3` or newer. Now install Composer with its official installer, which uses the PHP you just selected.

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
HASH="$(curl -sS https://composer.github.io/installer.sig)"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php --install-dir="$(brew --prefix)/bin" --filename=composer
rm composer-setup.php
composer --version
```

You should see `Installer verified`.

##### Install the application

Brew's NGINX serves from `$(brew --prefix)/var/www`, so put the application there.

```bash
mkdir -p "$(brew --prefix)/var/www"
cd "$(brew --prefix)/var/www"
composer create-project nishchay/nishchay nishchayApp
```

The installer asks for **your application name** and **your or your organisation's name**, then generates an encryption key and a service app id and secret. They are saved in `settings/configuration/`, and you can change them later.


##### Try it right away

You do not need a web server to see the application. PHP has a built-in one. Start it from inside the `public` folder.

```bash
cd "$(brew --prefix)/var/www/nishchayApp/public"
php -S localhost:8000
```

Open [http://localhost:8000](http://localhost:8000) and you should see the Nishchay welcome page. Press `Ctrl+C` to stop it. The built-in server is for trying things out, so continue below to set up NGINX for real use.

##### Create your site

Homebrew's NGINX reads every file in its `servers` folder and listens on port **8080**, which also means you do not need `sudo`. Create the file.

```bash
mkdir -p "$(brew --prefix)/etc/nginx/servers"
nano "$(brew --prefix)/etc/nginx/servers/nishchay-app.conf"
```

Paste the following, then save with `Ctrl+O`, `Enter`, `Ctrl+X`. **Replace `/opt/homebrew` with `/usr/local` on an Intel Mac**, or whatever `brew --prefix` prints.

```nginx
server {
    listen 8080;
    server_name app.nishchay.local;
    root /opt/homebrew/var/www/nishchayApp/public;
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
        fastcgi_pass 127.0.0.1:9000;
    }

    # Never serve hidden files such as .git.
    location ~ /\. {
        deny all;
    }
}
```

The root points at the application's **`public`** folder. Never point it at the project folder itself, because that would expose your settings.

PHP-FPM listens on `127.0.0.1:9000` by default. If another PHP version already uses that port, open `$(brew --prefix)/etc/php/8.4/php-fpm.d/www.conf`, change the `listen =` line to a free port, and use the same port in `fastcgi_pass`.

##### Start the services

```bash
brew services start php@8.4
nginx -t
brew services restart nginx
```

`nginx -t` checks the configuration and prints the file and line of any mistake. Both services start again after a restart of your Mac.

##### Add a host entry

This makes `app.nishchay.local` open the site on your own machine.

```bash
echo "127.0.0.1   app.nishchay.local" | sudo tee -a /etc/hosts
```

##### Check that it works

Open [http://app.nishchay.local:8080](http://app.nishchay.local:8080). You should see the Nishchay welcome page. The `:8080` is needed because Homebrew's NGINX listens there. If you change `listen 8080;` to `listen 80;` you can leave it out, but then start NGINX with `sudo brew services start nginx`.

##### If something is wrong

| What you see | What to do |
| ------------ | ---------- |
| **502 Bad Gateway** | PHP-FPM is not running or listens elsewhere. Run `brew services list`, and check that the `listen =` line in `www.conf` matches `fastcgi_pass`. |
| **404 on every page except `/`** | The `try_files $uri $uri/ /index.php?$args;` line is missing from `location /`. |
| **An error about your platform, or a PHP parse error** | The PHP version is too old. Run `php -v`, and check the PATH line from _Install PHP, NGINX and Composer_ is in `~/.zshrc`. |
| **Blank page or a permission error** | Run `ls -ld "$(brew --prefix)/var/www/nishchayApp/logs"`. Both services run as you when started with `brew services`, so the folder should be owned by your user. |
| **Failed opening required `./../bootstrap/init.php`** | You started the built-in server from the wrong folder. Start it from inside `public`. |
| **"Address already in use" for port 8080** | Another program uses it. Change `listen 8080;` here to a free port and use that port in the browser address. |

##### Next

- [Create your first controller and route](/learningCenter/getStarted/web)
- [Build a REST API](/learningCenter/getStarted/restApi)
- [Understand how a request is handled](/learningCenter/lifeCycle)
