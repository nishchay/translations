#### Route pattern

Route pattern works on controller method and its parameter. This makes route attribute to be optional if controller is following any predefined or custom pattern. Nishchay has few predefined route pattern which application can use. This patterns are listed below:

1. Action
2. Action method
3. Action method parameter
4. CRUD

Route pattern iterates over each method of controller and looks for its parameters, based on method name and its parameter routes are created.

##### Use pattern

To make controller follow route pattern, define it in `pattern` parameter of `Route` attribute.

```php

#[Route(prefix: 'user', pattern: 'action')]
class UserController {
```

Prefixing route is not required.

##### Action

Route pattern name: `action`

This pattern creates route if method name in controller starts with `action`.

```php

#[Route(prefix: 'user', pattern: 'action')]
class UserController {

    public function actionIndex(){

    }

    public function actionProfile() {

    }

    public function actionSetting() {

    }
}
```

Below routes will be created from above code:

1. `user`
2. `user/profile`
3. `user/setting`

Note that `actionIndex` results in `/` path. Route created using this pattern allows all HTTP methods. To define HTTP method use `actionMethod` route pattern.

##### Action method

Pattern name: `actionMethod`. This pattern creates route along with optional HTTP method. This pattern follows `action{?HTTP_METHODS}{RoutePath}`.

```php
public function actionGetUser() {

}
```

For above method `user` route will be created which will accept only `GET` request.

This pattern also allows creating route with multiple supported HTTP methods. To create route with multiple supported HTTP methods, it should be concated without space but HTTP method name should be in camel case only. See below example to understand how multiple http methods are created.

```php
public function actionGetPostUser() {

}
```

For above `user` route will be created which will accept both `GET` and `POST` http methods.

As HTTP method part is optional in this pattern, omitting it creates route which accepts any HTTP method. This is same as `action` pattern.

##### Action method parameter

Pattern name: `actionMethodParameter`. This pattern is an improvement to `actionMethod` patern allowing plceholder to be defined in route path using method parameters. Method parameter which has scaler type hint are considered placeholder segment for the route.

This pattern iterates over each method parameter for creating placeholder segment. It stops when it encounters non scaler type hint for the parameter. Below are supported scaler types:

1. String
2. Int
3. Bool
4. Array

```php

public function actionGetUser(int $userId) {

}
```

This creates `GET user/{userid}` where `userId` must be integer. When data type is boolean then placeholder value should be `1` or `0`. For data type array, list of items in an array results in `enum` type, this allows placeholder to have values only defined in an array.

###### Optional segment

When method parameter is nullable or it have default then it considered optional segment. Consider below example which create same route (`user/{userId}/{?action}`) path for both code.

```php
public function actionUser(int $userId, ?string $action) {
}

public function actionUser(int $userId, string $action = 'view') {
}

```

In the case of array data type, as its value considered as enum values which still makes placeholder required. To make array parameter as optional for the placeholder use nullable type on array data type.

##### CRUD

Pattern name: `crud`. This allow creating crud route based on method name only. When method name exactly matches below supported method name then it creates route associated with it.

| Method Name | Route   | HTTP Method |
| ----------- | ------- | ----------- |
| index       | `/`     | GET         |
| create      | `/`     | POST        |
| fetch       | `/{id}` | GET         |
| update      | `/{id}` | PUT         |
| delete      | `/{id}` | DELETE      |

For this pattern, method with name other than above are ignored. But still you can define route annotation and create route within this controller.

##### Pattern overriding

Although pattern creates route based on its route pattern being used. We can override this behaviour by defining route annotation on method. All predefined route pattern allows defining route annotation on method. To better understand how we can override route pattern let's start with basic route pattern `action`.

```php
public function actionUser() {

}
```

Above results in `@Route(path='user')` annotation. As this rotue is created with only parameter called `path`, we can override this parameter defining route with different path.

Overriding behaviour is different for each pattern, suppose for an example `actionMethod` pattern which creates route along with HTTP method if any defined on it. `@Route` annotation in this case gets created with two parameters `path` and `type` parameter. We can override any of these parameter or both of the parameter.

```php
public function actionGetUser() {

}
```

This results in `@Route(path='user',type=GET)` annotation. To have different path than above and keep the HTTP method as it is we can define rotue with only `path` parameter.

```php

#[Route(path: 'profile')]
public function actionGetUser() {

}
```

We can override only HTTP method by defining route with only `type` parameter.

##### User defined pattern

Apart from using predefined route pattern, we can define our pattern and then use in an application.

In `routes.php` setting file under `patterns` we can define list of route pattern to be used within application where key becomes name of route pattern.

Route pattern is created setting values of below configuration:

1. route
2. namedscope
3. service
4. response
5. processor

Based on value of above setting, we can create various kind of route pattern.

###### route

This setting is used of `@Route` annotation on controller method. If set value to `true` it will force controller to define route annotation to be defined on method.

Using `null` makes it optional.

###### Namedscope

Setting value of this to `true` forces route method to define `@namedscope` on it. This is useful if we are creating pattern and want to have all routes belongs to at least on scope.

We can also leave `namedscope` optional by setting value to `null`.

To disable `namedscope` to be defined on route method, set value to `false`.

We can specify list of scope which will be applied to all routes which follow created pattern. In some case we want all routes following our pattern to have only `namedscope` defined here and it should not override by any route. This can be done by setting value as shown below:

```php
[
    'name' => [List of scopes in array],
    'override' => false
]
```

###### service

This settings controls `@Service` annotation to make it required, optional or disabled.

1. Set `null` to make it optional
2. Set `true` to force it to be defined on method.
3. Set `false` to disallow annotation.

###### response

This setting allow controlling of what kind of response should be generated by route which follows created pattern.

We can configure pattern to allow only response which mentioned in this setting. To leave it to route, we can set it to `null` which will make `@Response` annotation optional.

To force annotation to be defined on method set value to `true`.

We can also set response type and make override disabled. by setting below setting:

```php
[
    'type' => 'json',
    'override' => false
]
```

###### processor

When pattern iterates over each method, this method get called with two arguments `className` and `methodName`.

This method should return route path as string or route annotation in an array.

###### Return route path only

```php
return 'pathName';
```

###### Return route annotation

```php
return [
    'path' => 'path',
    'type' => 'GET'
];
```

Notice that we can return route annotation with all supported parameters. Where key represents route annotation parameter name.

###### Return route and placeholder annotation

If prepared route contains placeholder segment then you must return `@Placeholder` along with `@Route` annotation.

```php
return [
    'route' => [
        'path' => 'user/{userId}/{action}'
    ]
    'placeholder' => [
        'userId' => 'int',
        'action' => 'string'
    ]
];
```

###### Ignore method

To ignore method to consider as route, return `false`.

###### Leave it to controller

To leave it controller method, return `null`. In this case controller method is considered as route only if it has `@Route` annotation defined on it.
