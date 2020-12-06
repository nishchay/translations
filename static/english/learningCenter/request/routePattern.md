#### Route pattern

Route pattern works on controller method and its parameter. This makes route annotation to be optional if controller is following any predefined or custom pattern. Nishchay has few predefined route pattern which application can use. This patterns are listed below:

1. Action
2. Action method
3. Action method parameter
4. CRUD

Route pattern iterates over each method of controller and looks for its parameters, based on method name and its parameter routes are created.

##### Use pattern

To make controller follow route pattern, define it in `pattern` parameter of `@Route` annotation.

```php
/**
* @Route(prefix=user,pattern=action)
*/
class UserController {
```

Prefixing route is not required.

##### Action

Route pattern name: `action`

This pattern creates route if method name in controller starts with `action`.

```php
/**
* @Route(prefix=user,pattern=action)
*/
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
/**
* @Route(path='profile')
*/
public function actionGetUser() {

}
```

We can override only HTTP method by defining route with only `type` parameter.

##### User defined pattern

Apart from using predefined route pattern, we can define our pattern and then use in an application.
