#### Install on Windows using Composer

There are two good ways to run Nishchay on Windows. Pick the one that fits.

| | Native PHP | WSL 2 with Ubuntu |
| --- | --- | --- |
| **Best for** | Trying the framework, and everyday development | Running a real web server (NGINX) that behaves like production |
| **You install** | PHP and Composer | Ubuntu inside Windows, then follow the Ubuntu guide |
| **Time** | About ten minutes | About twenty minutes |

> **Why not XAMPP?** The newest XAMPP download still ships PHP 8.2, and security support for PHP 8.2 ends on 31 December 2026. Nishchay supports current PHP versions only, so we no longer recommend it.

##### Before you start

- **PHP 8.3 or newer.** Older versions have reached end of life and are not supported. See the [supported PHP versions](https://www.php.net/supported-versions.php). The steps below use PHP 8.4.
- **Composer**, to download the framework and its dependencies.

---

##### Option 1: Native PHP

###### Install PHP

1. Open the [PHP for Windows download page](https://windows.php.net/download/) and download the **Non Thread Safe, x64** ZIP for PHP **8.4**.
2. Create the folder `C:\php` and extract the ZIP into it, so that `C:\php\php.exe` exists.
3. If you see an error about `VCRUNTIME140.dll` later, install the Microsoft Visual C++ Redistributable (x64) from Microsoft and try again.

###### Turn on the extensions

PHP for Windows starts with almost every extension switched off.

1. In `C:\php`, copy `php.ini-development` to a new file named `php.ini`.
2. Open `php.ini` in a text editor and remove the leading `;` from these lines. Some of them may already be active.

```ini
extension_dir = "ext"
extension=curl
extension=fileinfo
extension=gd
extension=mbstring
extension=openssl
extension=pdo_mysql
extension=zip
```

###### Put PHP on your path

1. Press the Windows key, type **environment variables** and open **Edit the system environment variables**.
2. Click **Environment Variables**, select **Path** under your user variables and click **Edit**.
3. Click **New**, add `C:\php` and click **OK** on every window.
4. Open a **new** Command Prompt and check:

```
php -v
php -m
```

`php -v` must say `PHP 8.3` or newer. `php -m` must list `curl`, `fileinfo`, `gd`, `mbstring`, `openssl`, `pdo_mysql` and `zip`.

###### Install Composer

Download and run [Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe). When it asks for the PHP command-line program, choose `C:\php\php.exe`. Then open a **new** Command Prompt and check:

```
composer --version
```

###### Install the application

Pick a folder for your projects, for example `C:\Projects`.

```
mkdir C:\Projects
cd C:\Projects
composer create-project nishchay/nishchay nishchayApp
```

The installer asks for **your application name** and **your or your organisation's name**, then generates an encryption key and a service app id and secret. They are saved in `settings\configuration\`, and you can change them later.

###### Run it

PHP has a built-in web server. Start it from inside the `public` folder.

```
cd C:\Projects\nishchayApp\public
php -S localhost:8000
```

Open [http://localhost:8000](http://localhost:8000) and you should see the Nishchay welcome page. Press `Ctrl+C` to stop the server.

The built-in server is meant for development. When you want a real web server, use Option 2.

---

##### Option 2: WSL 2 with Ubuntu

Windows Subsystem for Linux runs a real Ubuntu inside Windows, so you can follow the Ubuntu guide as it is written.

###### Install Ubuntu

1. Open **PowerShell as administrator**.
2. Run:

```
wsl --install -d Ubuntu
```

3. Restart Windows if it asks you to, then open **Ubuntu** from the Start menu and create your Linux user name and password.

###### Follow the Ubuntu guide

Inside the Ubuntu window, follow the [Ubuntu guide](/learningCenter/installation/ubuntu/composer) from the top. Two small differences:

- Keep the application in the Linux file system, in `/var/www` as the guide says. Do not put it under `/mnt/c`, because file access there is much slower.
- **Skip the guide's "Add a host entry" step.** It edits the Linux hosts file, but your browser runs on Windows. Instead:
  1. Press the Windows key, type **Notepad**, right-click it and choose **Run as administrator**.
  2. In Notepad choose **File > Open**, show **All files**, and open `C:\Windows\System32\drivers\etc\hosts`.
  3. Add this line at the end and save.

```
127.0.0.1   app.nishchay.local
```

WSL forwards the web server to Windows, so [http://app.nishchay.local](http://app.nishchay.local) opens in your Windows browser.

If `systemctl` reports that systemd is not running, open `/etc/wsl.conf` in Ubuntu (`sudo nano /etc/wsl.conf`), add the lines below, then run `wsl --shutdown` in PowerShell and open Ubuntu again.

```ini
[boot]
systemd=true
```

---

##### If something is wrong

| What you see | What to do |
| ------------ | ---------- |
| **`php` is not recognised** | Open a **new** Command Prompt after changing the Path. Windows only reads the Path when a window starts. |
| **`php -v` shows an old version** | Another PHP comes first in your Path. Move `C:\php` to the top of the list. |
| **Composer says an extension such as `mbstring` is missing** | Enable it in `C:\php\php.ini` as shown above, then open a new Command Prompt. `php --ini` shows which `php.ini` is in use. |
| **A message about `VCRUNTIME140.dll`** | Install the Microsoft Visual C++ Redistributable (x64). |
| **Failed opening required `./../bootstrap/init.php`** | You started the built-in server from the wrong folder. Start it from inside `public`. |
| **Port 8000 is already in use** | Pick another port, for example `php -S localhost:8080`. |
| **`composer` is not recognised** | Open a **new** Command Prompt after installing. |

##### Next

- [Create your first controller and route](/learningCenter/getStarted/web)
- [Build a REST API](/learningCenter/getStarted/restApi)
- [Understand how a request is handled](/learningCenter/lifeCycle)
