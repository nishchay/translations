#### Get started: REST API

In this tutorial you build a small API: a route that responds with JSON and lets the caller choose which fields it wants. It takes about ten minutes.

If you would rather start with a ready made API that can create, read, update and delete records, see [Your first resource](/learningCenter/getStarted/firstResource). Here you write the route yourself.

##### Before you start

You need a Nishchay application that runs on your machine. If you do not have one yet, open [Get Started](/learningCenter/getStarted), choose your system, and follow the installation guide it links to. It needs **PHP 8.3 or newer** and **Composer**.

To run the application while you work, start PHP's built-in server from the `public` folder of your application:

```bash
cd nishchayApp/public
php -S localhost:8000
```

Leave it running. Every address below uses `http://localhost:8000`. If you set up a web server and host name while installing, such as `http://app.nishchay.local`, use that instead.

Two ideas are enough to start. Requests are handled by a **route**. A route is a method of a **controller**, and a controller is a class that holds several routes.

For a route to get service features, such as tokens and response filtering, it also needs the `Nishchay\Attributes\Controller\Method\Service` attribute. You can make the whole application a service with the `enable` setting in `settings/configuration/service.php`. Here we only mark one route.

In your application, controllers live in `Application/Controllers`. Routes that are services belong in the `Service` folder, so we will use that.

##### Create a controller

A class becomes a controller when it has the `Nishchay\Attributes\Controller\Controller` attribute.

Create the file `Application/Controllers/Service/TestController.php` and paste this in.

```php
<?php

namespace Application\Controllers\Service;

use Nishchay\Attributes\Controller\Controller;

#[Controller]
class TestController
{

}
```

The namespace follows the folder the file is in.

##### Create a route

Add a method with the `Route` attribute, giving the `path` callers will use, and the `Service` attribute.

```php
<?php

namespace Application\Controllers\Service;

use Nishchay\Attributes\Controller\Controller;
use Nishchay\Attributes\Controller\Method\{Route, Service};

#[Controller]
class TestController
{

    #[Route(path: 'test')]
    #[Service]
    public function testRoute()
    {

    }

}
```

##### Respond with JSON

Declare the `Nishchay\Attributes\Controller\Method\Response` attribute to say the route responds with JSON. You can also make the whole application respond with JSON in `service.php`.

```php
use Nishchay\Attributes\Controller\Method\Response;

#[Route(path: 'test')]
#[Service(token: false)]
#[Response(type: 'JSON')]
public function testRoute()
{
    return [
        'name' => 'Bhavik Patel',
        'birth' => '01-01-2021',
        'gender' => 'Male',
        'country' => 'India',
        'relationship' => 'married'
    ];
}
```

The data is fixed here, only to show how a response looks. Remember to add `Response` to the `use` line at the top of the file.

Notice `token: false` in the `Service` attribute. By default every service needs an access token, so that only callers with a token can use it. We turn the token off here so you can try the route straight from a browser.

##### Check

Open [http://localhost:8000/test](http://localhost:8000/test) in a browser or an API client such as curl. It responds with:

```json
{
  "name": "Bhavik Patel",
  "birth": "01-01-2021",
  "gender": "Male",
  "country": "India",
  "relationship": "married"
}
```

##### Let the caller choose the fields

Services can filter their own response, and you do not need to change any code for it. A caller adds a `fields` parameter listing what it wants.

Open [http://localhost:8000/test?fields=name,gender](http://localhost:8000/test?fields=name,gender) and the response contains only **name** and **gender**.

##### Keep exploring

Services can do much more: protect routes with tokens, always return chosen fields, and use your own response format. Read about [web services](/learningCenter/service/webService), and [request handling](/learningCenter/request/requestHandling) for how routes work.
