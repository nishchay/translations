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

##### Route pattern