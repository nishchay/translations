#### Get started: Web application

In this tutorial you build a small page: a route that renders a view and greets a name you pass in the address. It takes about ten minutes.

If you want to see a whole resource working first, see [Your first resource](/learningCenter/getStarted/firstResource). Here you write the controller, the route and the view yourself.

##### Before you start

You need a Nishchay application that runs on your machine. If you do not have one yet, open [Get Started](/learningCenter/getStarted), choose your system, and follow the installation guide it links to. It needs **PHP 8.3 or newer** and **Composer**.

To run the application while you work, start PHP's built-in server from the `public` folder of your application:

```bash
cd nishchayApp/public
php -S localhost:8000
```

Leave it running. Every address below uses `http://localhost:8000`. If you set up a web server and host name while installing, such as `http://app.nishchay.local`, use that instead.

Two ideas are enough to start. Requests are handled by a **route**. A route is a method of a **controller**, and a controller is a class that holds several routes.

In your application, controllers live in `Application/Controllers`. That folder is split by how the routes are used:

| Folder | Routes in it |
| ------ | ------------ |
| `Direct` | Open straight from a browser. We will use this one. |
| `Secure` | Need a logged-in user. |
| `Service` | Web services and APIs. |

Views live in `Application/views`.

##### Create a controller

A class becomes a controller when it has the `Nishchay\Attributes\Controller\Controller` attribute.

Create the file `Application/Controllers/Direct/TestController.php` and paste this in.

```php
<?php

namespace Application\Controllers\Direct;

use Nishchay\Attributes\Controller\Controller;

#[Controller]
class TestController
{

}
```

The namespace follows the folder the file is in.

##### Create a route

A controller method is not a route until you say so. Add the `Nishchay\Attributes\Controller\Method\Route` attribute with the `path` people will type in the address.

```php
<?php

namespace Application\Controllers\Direct;

use Nishchay\Attributes\Controller\Controller;
use Nishchay\Attributes\Controller\Method\Route;

#[Controller]
class TestController
{

    #[Route(path: 'test')]
    public function testRoute()
    {

    }

}
```

##### Create a view

The route does not return anything yet. Whatever a route method returns is treated as the name of a view, and Nishchay renders that view as the response.

First create the view, `Application/views/test.twig`:

```twig
<h1>Hi Bhavik Patel</h1>
```

Then return its name from the route. The name is the file name without `.twig`.

```php
#[Route(path: 'test')]
public function testRoute()
{
    return 'test';
}
```

##### Check

Open [http://localhost:8000/test](http://localhost:8000/test). You should see **Hi Bhavik Patel** as a heading.

##### Pass data to the view

The name is fixed in the view. Let's take it from the address instead.

Change the route method so that it receives the name, and passes it on with `RequestStore`.

```php
use Nishchay\Http\Request\RequestStore;

#[Route(path: 'test')]
public function testRoute(?string $name)
{
    RequestStore::set('name', $name);
    return 'test';
}
```

The method parameter `$name` is filled from the **name** GET parameter. Because the type is nullable, `?string`, the parameter is optional.

`RequestStore` carries data to the view. Whatever you put in it is available there by the same name.

Now print it in `Application/views/test.twig`. Views use the [Twig](https://twig.symfony.com) template engine, so a variable is written with double curly braces.

```twig
<h1>Hi, {{ name }}</h1>
```

##### Check again

Open [http://localhost:8000/test?name=Jaggy Jones](http://localhost:8000/test?name=Jaggy%20Jones). You should see **Hi, Jaggy Jones**. Without `?name=` the name is empty.

##### Keep exploring

You have the basic loop: a controller, a route, a view and some data. Learn how [routes](/learningCenter/request/requestHandling) work in depth, how to read [request parameters](/learningCenter/request/requestParameter), then move on to [data handling](/learningCenter/data/entity).
