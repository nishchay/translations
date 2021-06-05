#### Request Handling

Requests are handled by controller method. Controller is just a class which can have various method to handle request, this method is known as route method. Any class which has `Controller` attribute is considered as controller class.

```php
use Nishchay\Attributes\Controller\Controller;

#[Controller]
class NamasteController {

}
```

`Nishchay\Attributes\Controller\Controller` is an attribute class. When its declared on class, class becomes controller. Within this controller we can create various method along with `Nishchay\Attributes\Controller\Method\Route` attribute on it. Any controller method which has this attribute are considered as route method.

##### Route

Controller methods are considered as route method if following conditions are met.

1. `Route` attribute must be declared on it.
2. It must belong to same class, method defined on parent class is not considered.
3. It must be public.
4. It must not be static.
5. It must not start with underscore.

###### Simple route

Below code demonstrates how we can create simple route.

```php
namespace Application\Controllers\Direct;

use Nishchay\Attributes\Controller\Controller;
use Nishchay\Attributes\Controller\Method\Route;

#[Controller]
class NamasteController {

    #[Route(path:'namaste')]
    public function namaste() {
        return 'namaste';
    }

}
```

Here we just created route called _namaste_ and it can be accessed by _{HOST}/namaste_. Suppose your application hosted at _http://app.nishchay.local_ then above route can access by _http://app.nishchay.local/namaste_

###### Setting landing/index route

Landing route is one where our application starts. It is the entry point of our application. When user hits only domain name, landing route is called. Landing route of application can changed by updating config _config.landingRoute_ which is in setting file _settings/application.php_.

###### Multiple segments

We can specify any valid string as route path. This path can be separated by **/** and each part is known as segment.

```php

#[Route(path:'user/messages')]
public function messageList() {
    return 'user/messages';
}
```

In above created route, there are two segments _user_ and _messages_.

###### Route with dynamic/placeholder segments

Segment can be dynamic, this segment is called placeholder segment. Placeholder segment can be any one of _string_, _number_, _int_ or _alphanum_. Placeholder segments are defined in opening _{_ and closed _}_ curly braces.

```php
use Nishchay\Attributes\Controller\Method\Route;
use Nishchay\Attributes\Controller\Method\Placeholder;

 #[Route(path:'user/messages/{userId}')]
 #[Placeholder(['userId' => 'int'])]
public function messageView() {
    return 'user/messages/view';
}
```

In above route path we have create route with path _user/messages/{userId}_ where _userId_ will be placeholder segment. What kind of _userId_ should have is defined using `Nishchay\Attributes\Controller\Method\Placeholder` attribute.

All placeholder segments defined in path must be passed as in in `Placeholder` attribute where key is segment name(In above case its _userId_) and its value should be type of segment.

We can define any number of placeholder segment in path. For each placeholder segment it should exists in `Placeholder` attribute parameter.

###### Optional segment

To make placeholder segment optional, prefix it by `?`. See below code for more understanding

```php

 #[Route(path='profile/?{userId}')]
 #[Placeholder(['userId' => 'int'])]
public function userProfile() {
    return 'user/profile';
}
```

Here _userId_ will become optional segment, so above route can be called with or without passing _userId_.

###### Choose method name as route name

We can make method name to considered as route path. This can be achieved by passing first parameter as _true_.

```php

#[Route(path:true)]
public function namaste() {
    return 'namaste';
}
```

Here route will be _namaste_.

###### Prefix all routes in controller

All routes defined under controller can be prefixed. Attribute `Nishchay\Attributes\Controller\Routing` allow us to create prefix for each router within controller.

```php

use Nishchay\Attributes\Controller\Controller;
use Nishchay\Attributes\Controller\Routing;

#[Controller]
#[Routing(prefix:'user')]
class User {

    #[Route(path:true)]
    public function profile() {
        return 'profile';
    }
}
```

Here actual route is created with path _profile_ which will be prefixed with _user_ and results in _user/profile_.

###### Ignore prefix defined by Controller

If controller has defined prefix, then each route defined within its controller are prefixed with it. But we can also ignore prefixing for specific route. This can be achieved by passing `Route` parameter `prefix: false`.

```php

#[Controller]
#[Routing(prefix:'user')]
class User {

    #[Route(path:true,prefix:false)]
    public function profile() {
        return 'profile';
    }
}
```

###### Prepare prefix from class name

If we want to prefix all routes within controller to be prefixed by class base name, this can achieved by specifying `prefix: 'this.base'`. Let's do some changes in above code.

```php

#[Controller]
#[Routing(prefix:'this.base')]
class User {

    #[Route(path:true)]
    public function profile() {
        return 'profile';
    }
}
```

Above code will create route _User/profile_.

Because class name's first character is in upper case, in above code route is prefixed with _User_. we can change the case of prefix with the help of `case` parameter which accepts below values.

1. lower: to change prefix in lower case. For above case, prefix will be _user_.
2. upper: to change prefix in upper case. This will make prefix in above case to _USER_.
3. camel: to change prefix in camel case. This actually makes first character lower case and keeps remaining character same.

##### Route HTTP methods

Route can also be created for single or multiple http methods. By default its for all http method. We can define http method for route with the help of `type` parameter of `Route` attribute. Value of it can be string or array(To make route to handle multiple HTTP methods).

```php

#[Controller]
class User {

    #[Route(path:'profile',type:'GET')]
    public function profile() {
        return 'profile';
    }
}
```

Here _profile_ path will work for GET http method only. Because we have defined _profile_ route for _GET_ http method, we can also define same route name for remaining http method also.

```php

#[Controller]
class User {

    #[Route(path:'profile',type:'GET')]
    public function profile() {
        return 'profile';
    }

    #[Route(path:'profile',type:'POST')]
    public function saveProfile() {
        return 'profile';
    }
}
```

When we hit url _profile_ with _GET_ HTTP method, `User::profile` method will be called and for HTTP method _POST_ method `User::saveProfile` will be called.

If we don't define any HTTP method for the route, it will have all HTTP method, defining same route again with other HTTP method will result in an error. See below code:

```php

#[Controller]
class User {

    #[Route(path:'profile')]
    public function profile() {
        return 'profile';
    }

    #[Route(path:'profile',type:'POST')]
    public function saveProfile() {
        return 'profile';
    }
}
```

Above will result in an exception because _profile_ method is handling all type of HTTP method for path _profile_ and _saveProfile_ method is again declaring _profile_ path with _POST_ HTTP method which is duplicated.

Remember route path with HTTP method must be unique for application. Defining duplicate route will result in an exception.
When we define any route, please make sure other route with route name and HTTP method already exists. If there is conflict, please check that other route is created for specific HTTP method only.

##### Route priority

Because routes are processed how they are defined in an application, its stored with same priority in route collection.
This will create problem if we have defined route which have dynamic segment.

```php

#[Controller]
class Post {

    #[Route(path:'post/{postname}',type:'GET')]
    #[Placeholder(['postName' => 'string'])]
    public function viewPost() {
        return 'post/view';
    }

    #[Route(path:'post/create',type:'GET')]
    public function createPost() {
        return 'post/create';
    }

    #[Route(path:'post/help',type:'GET')]
    public function postHelp() {
        return 'post/help';
    }
}
```

Here Nishchay will stores route in collection in following order, these routes are stored with same order of its class.

1. GET post/{postName}
2. GET post/create
3. GET post/help

If we hit url _{HOST}/post/create_, what we expect that _post/create_ should be called but it won't get called because when nishchay processor will find route, it will be first matched with _post/{routeName}_. This is because it was first stored in collection.

We can give priority to routes and route with higher priority will be stored first in route collection. Let's change above code:

```php

#[Controller]
class Post {

    #[Priority(value:2000)]
    #[Route(path:'post/{postname}',type:'GET')]
    #[Placeholder(['postName' => 'string'])]
    public function viewPost() {
        return 'post/view';
    }

    #[Priority(value:3000)]
    #[Route(path:'post/create',type:'GET')]
    public function createPost() {
        return 'post/create';
    }

    #[Priority(value:4000)]
    #[Route(path:'post/help',type:'GET')]
    public function createPost() {
        return 'post/help';
    }
}
```

Now above routes will be stored in following orders, Route with higher will be stored first.

1. GET post/help
2. GET post/create
3. GET post/{postName}

**NOTE:** Route's default priority is: 100

##### Stage route

An application can have multiple stages like _local_, _test_ and _live_. Application stage is defined in _settings/applicationl.php_ file. We can create route for _local_ and _test_ application stage. This can be defined using `stage` parameter of `Route` attribute. Stage route is not applicable for _live_ application stage.

If we have defined route for _test_ stage then that route will be available for application stage _test_ only. See below example:

```php

#[Route(path:'checkTest',stage:'test')]
public function testRoute() {

}
```

Above will create _checkTest_ route applicable for only _test_ application stage. If application is _local_ stage then this route won't be available.

Route can also be defined for both _local_ and _test_ stage, by passing array _['local','test']_ to `stage` parameter.

##### Abstract/Static route

In an application there can be pages which displays only static contents, such pages are known as about us, terms and conditions and help pages and there can be many more. Because these pages are static, it only needed to render only view. In this case instead of using normal route, we can define route on abstract method which will then act as abstract(static) route.

```php

#[Route(path:'help')]
abstract public function help();
```

Which view should be rendered is decided based on route path or `view` parameter of `Response` attribute. As in the above code, there's no response attribute defined. In this case route path will be taken as view path. When route path is taken as view path it can be prefixed by setting _abstractViewPath_ setting of _response.php_. This will be applicable to all abstract route where route path is taken as view path.

If we don't want to take route path as view path then it should be defined in `view` parameter of `Response` attribute.

We recommend using interface for creating abstract routes.

Advantage of using abstract route is that, nishchay don't need create instance of controller and so no need to call route method.

##### Redirect request

When we need to redirect request, route must return instance of `RequestRedirector`. We can either create instance of `RequestRedirector` or use `Request::getRedirectWithin` to redirect within application. If want to redirect outside of application use `Request::getRedirect`. All of these accept only one argument which should be route path or url in case of redirecting outside of an application.

Redirecting using `Request::getRedirectWithin`

```php

#[Route(path:true)]
public function admin() {
    if ($this->isUserAdmin() === false) {
        return Request::getRedirectWithin('unauthorized');
    }
    ...
}
```

In above code we want to make sure that user without admin access can not visit admin page. If user is not admin, we will redirect user to _unauthorized_ page which will tell user that user is not authorized to access this.

Above code can be implemented with the help of `RequestRedirector`

```php

#[Route(path:true)]
public function admin() {
    if ($this->isUserAdmin() === false) {
        return new RequestRedirector('unathorized');
    }
    ...
}
```

Redirecting outside application

```php

#[Route(path:true)]
public function redirectTo() {
    $url = Request::get('to'); # This will return value GET parameter name: to
    if (UrlChecker::isUrlSafe($url)) {
        return Request::getRedirect($url)
    }

    return Request::getRedirectWithin('unsafeUrl');
}
```

Above route is to redirect to any url which is outside of our application. This will check redirecting url is safe or not. `UrlChecker` is custom class which we can create to check url. If url is safe, we will redirect user to requested url otherwise we will redirect to _unsafeUrl_ which will inform user that url is not safe.

##### Forward request

Nishchay provides feature to forward request internally. When request is forwarded to another route, Client won't realize that it was forwarded. This is because it happens on server side only.

```php

#[Route(path:true)]
public function register() {
    return (new RequestForwarder('login'));
}

#[Route(path:true)]
public function login() {
    return (new RequestRedirector('profile'));
}
```

What will happen here is that user is first registered. Once user registered we will forward request to login route so that user logged into account. Once logged in, we will redirect user to their profile page.

Redirect changes url in browser while forward does not, forward happens in one request only.

Every time request is forwarded it creates new stage in request. Stage number starts from 1, increased by 1 for each forward. We can get stage detail of request using `Processor::getStageDetail('detailName')`.

We can redirect url with HTTP method _GET_ only but in forward we can forward with any HTTP method including POST parameters.

###### Set request type

By default request is forwarded with _GET_ HTTP method and it can changed to valid request method by passing it in second argument.

```php
(new RequestForwarder('routePath', 'POST'))
```

###### GET parameter

By passing array of values in `withGetParameter`, will forward request with GET parameter.

```php
    (new RequestForwarder('routePath'))
                ->withGetParameter(['param1'=>'value1','param2'=>'value2'])
```

###### POST parameter

We can also forward request with POST parameter. Passing POST parameter for another request method type is ignored.

```php
(new RequestForwarder('routePath', 'POST'))
            ->withPostParameter(['param1'=>'value1','param2'=>'value2'])
```

###### With flushing request store

Data added to request store are available for each stages of request. But we can remove all added data while forwarding request to next route.

```php
(new RequestForwarder('routePath'))
            ->withFlushRequestStore();
```

Just calling `withFlushRequestStore` removes all request store data on forwarding request.

##### Prevention on Forwarding request

We can put prevention on forwarding request from one route to another route. We can also prevent request to be forwarded to specific route.

- **Ascent**: Prevent request to be forwarded to route.
- **Descent**: Prevent request to forwarded from route.

Below code will prevents any request to be forwarded to this route.

```php

     #[Route(path:'routePath')]
     #[Forwarder(ascent:false)]
    public function methodName() {

    }
```

Attempt to forward request to this route results in exception. Just like ascent we can also prevent request to be forwarded from route.

```php

     #[Route(path:'routePath')]
     #[Forwarder(descent:false)]
    public function methodName() {

    }
```

##### Prevent incoming request / Create Private route

We can make route private so that it can used for internal only. By setting parameter `incoming: false` of `Route` attribute will prevent any incoming request. This is useful when we need to create route that can be used for forward request only and this can also be used in CRON job.

```php

     #[Route(path:'routePath',incoming:false)]
    public function methodName() {

    }
```

##### Required GET/POST parameter

Route might need some GET/POST parameter, Nishchay has attribute which verifies if required parameter exist. If required parameter does not found it takes action based on attribute declaration.

```php
use Nishchay\Attributes\Controller\RequiredGet;

#[Route(path:'routePath')]
#[RequiredGet(parameter:['param1'])]
public function methodName() {

}
```

Calling route path without GET parameter _param2_ will result in _Route not found_ exception. We can specify any number of required parameter for route.

`Nishchay\Attributes\Controller\RequiredGet` attribute is for requiring GET parameter and `Nishchay\Attributes\Controller\RequiredPost` attribute is for requiring POST parameter.

We can also define both of these annotations on controller class and it will be applied to all routes within controller.

##### Only GET/POST parameter

Only GET/POST parameter means route will require exact parameter. There must not be any parameter missing from parameter list and also there must not be extra parameter.

```php
use Nishchay\Attributes\Controller\OnlyGet;

#[Route(path:'routePath')]
#[OnlyGet(parameter:['a','b'])]
public function methodName() {

}
```

Above route will be called only if it is requested with exact parameter mentioned in `OnlyGet` attribute. Below is list of request which will be valid or not.

| Request               | Result                                         |
| --------------------- | ---------------------------------------------- |
| routePath?a=1&b=2     | Valid                                          |
| routePath             | Fail as there are not parameter                |
| routePath?a=1         | Fails because parameter `b` is missing         |
| routePath?a=1&b=2&c=3 | Fails because parameter `c` is extra parameter |

We can also define same for POST parameter using `Nishchay\Attributes\Controller\OnlyPost` attribute which have parameter same as `OnlyGet` attribute. These attributes can also be declared on controller class to apply to all routes within controller.

##### Scope

Defining single or multiple scope helps in events, sessions and exception handling. Using `Nishchay\Attributes\Controller\Method\NamedScope` attribute we can define single or multiple scope for the route.

```php

#[Route(path:true)]
#[NamedScope(name:'secure')]
public function profile() {

}
```

Above code will create named scope _secure_ for route _profile_. We can also define multiple scope for route.

When we define multiple scope for route then by default first scope in list becomes default scope of route. We can also specify default scope for the route with the help of `default` parameter.

To understand when route scope is useful, see below code.

```php

#[Route(path:true)]
#[NamedScope(name:'secure')]
public function profile() {
    # To view user profile
}


#[Route(path:'profile/setting')]
#[NamedScope(name:'secure')]
public function profileSetting() {
    # For profile settings
}

#[Route(path:true)]
#[NamedScope(name:'secure')]
public function messages() {
    # To view messages

```

Now we have multiple route for the profile and its related activity, we also have to make sure that user without login must not access above defined route.

To do this we can define event for `secure` scope within which we can check if user is logged or not. If user is not logged we can redirect user to login page. Please learn more about [events](/learningCenter/events/events).

Scope is also used in following features

1. [Session](/learningCenter/session), Learn how to define session for scope
2. [Exception handling](/learningCenter/exceptionHandling), Learn how to define exception for scope

##### Events

Events can be defined on controller or method. Events defined on controller will be executed for each routes of controller class. These events can be executed before or after execution. There are two attributes `Nishchay\Attributes\Controller\Event\BeforeEvent` and `Nishchay\Attributes\Controller\Event\AfterEvent`, both of these can be declared on controller or method.

Both of event attribute supports following parameter:

| Parameter | Description                                                                                                                                                                |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| callback  | Callback method name. This can be only method name if its within same controller or specify with `{class}::{method}` for callback outside of controller                    |
| order     | Order of execution. By default events are executed in [global, context, scope] order. This order can be changed using this parameter                                       |
| once      | Whether to execute event only once during request. If the route is forwarded then event may execute multiple time. This can be prevented if we set this parameter to true. |

```php

#[Route(path:true)]
#[BeforeEvent(callback:'checkLogin')]
public function profile() {
    # View profile
}
```

###### Event response

Event must return _true_ to denote its success. If event is success then next action is taken. If there are no further events then route will be executed otherwise next event will be executed.

When event responds with _false_, request will result in an exception `BadRequestException`. If event respond with other than `boolean` type, it will taken as request response. We can return view name or we can also redirect request. We can also forward request from event.

Even if route returns view or we redirect request, after event is always executed. On failure of event does not execute after event.

##### Stages of request

Upon each request, stages starts from number 1 which keep increases by 1 count for each forward request. Each stage store details related to current request for the stage.

###### Get request/stage detail

We can get following detail of stage using `Processor::getStageDetail` method. `Processor` is facade class, in use section it should be included with `use Processor` statement. It changes on each request forward.
| Request | Result |
| ----------- | ----------- |
| urlString | path of request. Empty when accessed only domain name. |
| urlParts | Url string divided by `/` |
| mode | Stage mode. Initial is always incoming, it changes to `forward` if request if forwarded. For maintenance mode its changed to `maintenance`. |
| stage | Stage number of request. Increased by 1 for each forward. |
| object | Instance of `Route` attribute of current request. |
| priority | Priority value of current route. |
| urlex | Route path with regular expression. When we define route with dynamic segment it will be converted regex. |
| segment | List of special segment in an array. |
| context | Context name |
| scopeName | Scope name of route. |
