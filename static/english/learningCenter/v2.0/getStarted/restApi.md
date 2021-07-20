#### Get Started: REST API

Before we proceed for get started tutorial for web application. You should understand some basic concept of framework. If you are familiar with these concepts than you can jump to _Create controller_ section.

In a framework, requests are handled by route. This route is method of controller. Controller is class which contains multiple routes.

In order for route to have service related feature, route method also need `Nishchay\Attributes\Controller\Method\Service` attribute to be declared. We can also make whole application as services by setting `enable= true` in _service.php_ file. But for now we just want to get started with REST API, we will here just show how basic REST API can be developed in **nishchay**.

##### Create controller

Controller is class which can have multiple route methods. We will start by creating class and than we will make it controller. This can be done by declaring `Nishchay\Attributes\Controller\Controller` attribute on class.

Create file called **TestController.php** in controller directory and paste below code.

```php
use Nishchay\Attributes\Controller\Controller;

#[Controller]
class TestController {

}
```

##### Create route

Now we will create route in above controller. We will do this by making changes in above code.

```php
use Nishchay\Attributes\Controller\Controller;
use Nishchay\Attributes\Controller\Method\Route;
use Nishchay\Attributes\Controller\Method\Service;

#[Controller]
class TestController {

    #[Route(path: 'test')]
    #[Service]
    public function testRoute() {

    }

}
```

##### Respond with JSON data

To make route respond with JSON data, we have to declare `Nishchay\Attributes\Controller\Method\Response` attribute on route method(We can also make whole application to respond with JSON using _service.php_ config file). See below below code:

```php
use Nishchay\Attributes\Controller\Method\Response;

#[Route(path: 'test')]
#[Service(token:false)]
#[Response(type: 'JSON')]
public function testRoute() {
    return [
        'name' => 'Bhavik Patel',
        'birth' => '01-01-2021',
        'gender' => 'Male',
        'country' => 'India',
        'relationship' => 'married'
    ];
}
```

We here have just added hardcoded response just to show how JSON data can be responded with REST API. Also note that we have `token: false` in `Service` attribute, this is because by default all APIs are marked as token required. This makes API to be accessed using only _token_.

But for now we just want to check we have disabled token for the above API.

##### Check

Let's now check what we did till now working perfectly. We can check this by visiting created route from browser or REST API client. Suppose we have created our web application at host _http://app.nishchay.local_ then hit _http://app.nishchay.local/test_.

Hitting above URL will respond with:

```json
{
  "name": "Bhavik Patel",
  "birth": "01-01-2021",
  "gender": "Male",
  "country": "India",
  "relationship": "married"
}
```

##### Service features

Nishchay has many service related features but for now we will look at response filtering. For this to check we don't have to make any changes in above code because all service features are applied by default.

If we hit service with URL `test` it respond with all data. We can filter responded data by sending fields parameter.

Hit service with _test?fields=name,gender_ and it will respond with only _name and gender_ fields.

There are many other feature for service, which you can learn [here](/learningCenter/service/webService).
