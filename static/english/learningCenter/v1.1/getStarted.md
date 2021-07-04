#### Get Started.

**NOTE**: This guide uses structure which comes with installation.

In this getting started step, we will create route which handles request sent by client and created route respond with view. Once you finish, You will have a basic idea about following things.

* Installation
* Open Structure
* Annotations
* Creating routes
* Creating views
* Passing data view
* Application settings

##### Install Nishchay

Nishchay can only be installed using composer only. If you don't have it, please download from [here](https://getcomposer.org/download).

Go to the directory where you host all your application and fire below command to install Nishchay.
```
composer create-project nishchay/nishchay learningNishchay
```

This will install Nishchay inside learningNishchay directory.

##### Create Route

To create route, you first need to create method in controller class. After that create [@route](/learningCenter/annotations/request/route) annotation on method. Code example is shown below.
```php
/**
* @Route(path='getStarted')
*/
public function getStarted(){
    // Implementation
}
```
Controller class is located at `Application/Controllers/Direct/NamasteController.php`.

We have created route called `getStarted`. Method `getStarted` will be treated as request handler.
If request matches with `getStarted`, above method will be executed.

##### Create view

We have created route, but we need to create view so that our route can generate response for request. Create a file `getStarted.php` in `Application/views/direct`.

Paste below code to view file.
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
    </head>
    <body>
        Wow <?= $name ?> you did it well, you created your first route.
    </body>
</html>
```
##### Make route to respond with view

We have created route and view. But we have to make route to respond with view. To do that just return view name from route method.

Your updated route will look like:

```php
/**
* @Route(path='getStarted')
*/
public function getStarted(){
    return "direct/getStarted";
}
```

##### Pass data to view

We are sure you have noticed that, in view there is variable called `$name` which is getting printed. We will send `$name` from our route by using `Nishchay\Http\Request\RequestStore` class. This class is used for passing data to view and it can also be used at any other places too.

RequestStore has method called `add` which allows us to store data to request store. This method accepts two parameter key and it's value. Anything you add to request store, will be converted to variable for your view. This means you can use key name as variable inside your view.

So we will update our route code by adding `RequestStore::add('name',YOUR_NAME)` above return statement. Please replace `YOUR_NAME` with your name. Once you add it, make sure your code should be similar to
```php
/**
* @Route(path='getStarted')
*/
public function getStarted(){
    RequestStore::add('name',YOUR_NAME);
    return 'direct/get-started';
}
```

Remember to add `use Nishchay\Http\Request\RequestStore;` before class declaration.

##### Set landing route of the application

Application home page is known as landing route. If client requests domain name only, landing route will be executed. This setting is available in `settings/configuration/application.php` file. You have to change the value of `route.landing`. Change this value to `getStarted`

##### Host your application locally

You need to host your application so that it can accessed. This depends on which server you use. You can use any of following or other servers. Below servers are mostly used server:

1. Nginx
2. Apache
3. IIS

There's XAMPP, LAMP and WAMP which comes with apache server, mysql and php, so you don't have to install all required dependencies separately.

Below is configuration file for `nginx` server:

```conf
server {
    listen 80;
    listen [::]:80;
    server_name app.nishchay.local;
    root   '/var/www/nishchayApp/public/';
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

##### Document root

Document root must be `public/index.php`.

##### All done.

Now we are done. Please hit the url(`http://app.nishchay.local`) on your browser. You should be able to see below response.

```html
Wow {YOUR_NAME_WILL_APPEAR_HERE} you did it well, you created your first route.
```
