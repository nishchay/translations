#### Get started: Web

Before we proceed for get started tutorial for web application. You should understand some basic concept of framework. If you are familiar with these concepts than you can jump to _Create controller_ section.

In a framework, requests are handled by route. This route is method of controller. Controller is class which contains multiple routes. We have dedicated documentation for this but for now this much understanding is enough for getting started with **nishchay**.

##### Create controller

Controller is class which can have multiple route methods. We will start by creating class and than we will make it controller. This can be done by declaring `@Controller` annotation on class.

Create file called **TestController.php** in controller directory and paste below code.

```php
/**
* @Controller
*/
class TestController {*

}
```

##### Create route

Now we will create route in above controller. We will do this by making changes in above code.

```php
/**
* @Controller
*/
class TestController {

    /**
    * @Route(path=test)
    */
    public function testRoute() {

    }

}
```

All methods in controller are not considered as route by default. To make controller method a route, we have to declare `@Route` annotation on method. When we declare this annotation on method, we must pass `path='ROUTE_PATH'`. This path can be used in url to access this route.

##### Create view

We have create route but currently route is not returning anything, we will now change above method and return view name so that when we call above route, view will be rendered in response.

Let's first create `test.twig` file in **views** directory.

```twig
<h1>Hi Bhavik Patel</h1>
```

and then making changes in route method.

```php
/**
* @Route(path=test)
*/
public function testRoute() {
    return 'test';
}
```

By default whatever returned by route method is considered as view name. Nishchay then locates returned view name and renders it for response.

##### Check

Let's now check what we did till now working perfectly. We can check this by visiting created route from browser. Suppose we have created our web application at host _http://app.nishchay.local_ then hit _http://app.nishchay.local/test_.

Hitting above URL in browse should print **Hi Bhavik Patel** in `H1` tag.

##### Bit improvements - Pass data to view

In above code we have added hardcoded name in view. We are now going change it so that name can be passed in GET parameter and it will be rendered in view.

First change `testRoute` method.

```php
/**
* @Route(path=test)
*/
public function testRoute(?string $name) {
    RequestStore::set('name', $name);
    return 'test';
}
```

Here method parameter we have added `$name` parameter which will have value from **name** GET request parameter. We also made string type to _nullable_ so that name request parameter will be optional.

`RequestStore` is used for passing data to view. Whatever we add in request store will be available in view.

Let's now change our view file to print `$name` passed from controller.

```twig
<h1>Hi, {name}</h1>
```

##### Check

If we hit URL with _?name=Jaggy Jones_ then **Hii Jaggy Jones** will be printed.

##### Keep exploring

You have now completed basic tutorial for getting started with framework. Nishchay has many features which will help you many ways. You can start by learning [Route](/learningCenter/request/requestHandling). Once that is done start learning [Data handling](/learningCenter/data/entity).
