#### Request Handling
Nishchay has controller class which contains all routes to handle request. Routes are only contained in controller class. To do this we first need to create controller by annotating class with `@Controller` annotation and then we can defined routes within it. Below code shows simple route.

##### Route
Routes are defined using `@Route` and it should follow below requirements.

1.  It must belong to same class, method defined on parent class is not considered.
2.  It must be public.
3.  It must not be static.
4.  It must not start underscore.
5.  @Route annotation must have one of parameter set `path` or `see`.

###### Simple route
Nishchay allows creating route path with any valid string.
```php
namespace Application\Controllers\Direct;

/**
* @Controller
*/
class NamasteController {
    /**
    * @Route(path='namaste')
    */
    public function namaste() {
        return 'namaste';
    }
}
```
Here we just created route called `namaste` and it can be accessed by `{HOST}/namaste`.

###### Setting landing/index route
Landing route is one where our application starts. It is the entry point of our application. When user hits only domain name, landing route is called. Landing route of application can changed by updating config `config.landingRoute` which is in setting file `settings/application.php`.

###### Multiple segments
Route created above has only one segment. Route path is divided by `/`, each part is known as segment.
```php
/**
 * @Route(path='user/messages')
 */
public function messageList() {
    return 'user/messages';
}
```
###### Route with dynamic/placeholder segments
Segment which has dynamic value is called placeholder segment. Placeholder segment can be any valid string or int.
```php
/**
 * @Route(path='user/messages/{userId}')
 * @Placeholder(userId=number)
 */
public function messageView() {
    return 'user/messages/view';
}
```
Here `userId` is called placeholder segment which should be number as defined `@Placeholder` annotation parameter `userId`. If route has segment with opening and closing brace, it is counted as placeholder segment. If route has placeholder segment, it requires `@Placeholder` annotation defined on route and `@placeholder` annotation must contain parameter name same as placeholder segment of route.

As per above code, Nishchay check for `@Placeholder` annotation is defined for route or not. It also check for each placeholder segment to be exist as parameter name in `@Placeholder` annotation. Placeholder segment type can be from one of these `string`, `number` and `alphanum`.

###### Optional segment
To make placeholder segment optional, prefix it by `?`. See below code for more understanding
```php
/**
 * @Route(path='profile/?{userId}')
 * @Placeholder(userId=number)
 */
public function userProfile() {
    return 'user/profile';
}
```
Here `userId` will become optional segment. Route can have any number of optional segment.

###### Choose method name as route name
We can make method name to considered as route path. This can be achieved by setting parameter `see = true`. When `see` is `true`, `path` parameter is ignored. `see` parameter has higher priority then `path` parameter.
```php
/**
 * @Route(see=true)
 */
public function namaste() {
    return 'namaste';
}
```
Here route will be `namaste`.

###### Prefix all routes in controller
All routes defined under controller can be prefixed. Controller supports `@Routing` annotation which allow us to create prefix for each router within controller.
```php
/**
* @Routing(prefix = 'user')
* @Controller
*/
class User {

    /**
    * @Route(see=true)
    */
    public function profile() {
        return 'profile';
    }
}
```
Here actual route is created with path `profile` which will be prefixed with `user` and results in `user/profile`.

###### Ignore prefix defined by Controller

If controller has defined prefix, then each route defined within its controller are prefixed with it. But we can also ignore prefixing for specific route. This can be achieved by setting `@Route`'s parameter `prefix = false`.

```php
/**
* @Routing(prefix='user')
* @Controller
 */   
class User {

    /**
    * @Route(see=true,prefix=false)
    */
    public function profile() {
        return 'profile';
    }
}
```

###### Prepare prefix from class name
If we want to prefix all routes within controller to be prefixed by class base name, this can achieved by specifying `prefix='this.base'`. Let's do some changes in above code.

```php
/**
* @Routing(prefix = 'this.base')
* @Controller
 */   
class User {

    /**
    * @Route(see=true,ignore=true)
    */
    public function profile() {
        return 'profile';
    }
}
```

Above code will create route `User/profile`.

Because class name's first character is in upper case, in above code route is prefixed with `User`. we can change the case of prefix with the help of `case` parameter which accepts below values.

1. lower: to change prefix in lower case.
2. upper: to change prefix in upper case.
3. camel: to change prefix in camel case. This actually makes first character lower case and keeps remaining character same.

###### Route HTTP methods

Route can also be created for single or multiple http methods. By default its for all http method. We can define http method for route with the help of `type` parameter of `@Route` annotation. Value of it can be string or array.

```php
/**
* @Controller
 */   
class User {

    /**
    * @Route(path='profile',type=GET)
    */
    public function profile() {
        return 'profile';
    }
}
```

Here `profile` path will work for GET http method only. Because we have defined `profile` route for `GET` http method, we can also define same route name for remaining http method also.

```php
/**
* @Controller
 */   
class User {
    /**
    * @Route(path='profile',type=GET)
    */
    public function profile() {
        return 'profile';
    }

    /**
    * @Route(path='profile',type=POST)
    */
    public function saveProfile() {
        return 'profile';
    }
}
```

When we hit url with `GET profile`, `User::profile` method will be called and for HTTP method `POST` method `User::saveProfile` will be called.

If we don't define any HTTP method for the route, it will have all HTTP method, defining same route again with other HTTP method will result in an error. See below code:

```php
/**
* @Controller
 */   
class User {
    /**
    * @Route(path='profile')
    */
    public function profile() {
        return 'profile';
    }

    /**
    * @Route(path='profile',type=POST)
    */
    public function saveProfile() {
        return 'profile';
    }
}
```

Here we will get error second method `saveProfile` because combination of route and HTTP method is already created.

Remember route path with HTTP method must be unique for application. Defining duplicate route will result in an error.
When we define any route, please make sure other route with route name and HTTP method already exists. If there is conflict, please check that other route is created for specific HTTP method only.

##### Route priority
Because routes are processed how they are defined in an application, its stored with same priority in route collection.
This will create problem if we have defined route which have dynamic segment, this can match if it was stored first in route collection. Let's take below example for more understanding

```php
/**
* @Controller
 */   
class Post {
    /**
    * To view post
    *
    * @Route(path='post/{postName}', type=GET)
    * @Placeholder(postName=string)
    */
    public function viewPost() {
        return 'post/view';
    }

    /**
    * This will print form for creating new post.
    *
    * @Route(path='post/create',type=GET)
    */
    public function createPost() {
        return 'post/create';
    }

    /**
    * This route is for post help.
    *
    * @Route(path='post/help',type=GET)
    */
    public function createPost() {
        return 'post/help';
    }
}
```

Here Nishchay will stores route in collection with following, these routes are stored with same order of its class.

1. GET post/{postName}
2. GET post/create
3. GET post/help

If we hit url `{HOST}/post/create`, what we expect that `post/create` should be called but it won't get called because when nishchay processor will find route, it will be matched first with `post/{routeName}`. This is because it was first stored in collection.

We can give priority to routes and route with higher priority will be stored first in route collection. Let's change above code:

```php
/**
* @Controller
 */   
class Post {
    /**
    * To view post
    *
    * @Priority(value=2000)
    * @Route(path='post/{postName}', type=GET)
    * @Placeholder(postName=string)
    */
    public function viewPost() {
        return 'post/view';
    }

    /**
    * This will print form for creating new post.
    *
    * @Priority(value=3000)
    * @Route(path='post/create',type=GET)
    */
    public function createPost() {
        return 'post/create';
    }

    /**
    * This route is for post help.
    *
    * @Priority(value=4000)
    * @Route(path='post/help',type=GET)
    */
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

##### Redirect request
When we need redirect request, route must return instance of `RequestRedirector`. We can either create instance of `RequestRedirector` or use `Request::getRedirectWithin` to redirect within application. If want to redirect outside of application use `Request::getRedirect`. All of these accept only one argument which should be route.

When we use `RequestRedirector` or `Request::getRedirectWithin` use only route name without host name. For `Request::getRedirect` pass http url.

Redirecting using `Request::getRedirectWithin`

```php
/**
* @Route(see=true)
*/
public function admin() {
    if ($this->isUserAdmin() === false) {
        return Request::getRedirectWithin('unathorized');
    }
    ...
}
```

As per above, we want to make sure that user without admin access can not visit admin page. If user is not admin, we will redirect route to `unauthorized` page which will tell user that user is not authorized to access this.

Above code can be implemented with the help of `RequestRedirector`

```php
/**
* @Route(see=true)
*/
public function admin() {
    if ($this->isUserAdmin() === false) {
        return new RequestRedirector('unathorized');
    }
    ...
}
```

Redirecting outside application

```php
/**
* @Route(see=true)
*/
public function redirectTo() {
    $url = Request::get('to'); # This will return value GET parameter name: to
    if (UrlChecker::isUrlSafe($url)) {
        return Request::getRedirect()
    }

    return Request::getRedirectWithin('unsafeUrl');
}
```

Above route is to redirect to any url which is outside of our application. This will check redirecting url is safe or not. `UrlChecker` is custom class which we can create to check url. If url is safe, we will redirect user to requested url otherwise we will redirect use `unsafeUrl` which will tell user that url is not safe.


##### Forward request
Nishchay provides feature to forward request internally. When request is forwarded to another route, Client won't realize that it was forwarded. This is because it happens on server side only.

```php
/**
 * @Route(path='register')
 */
public function register() {
    return (new RequestForwarder('login'));
}

/**
 * @Route(path='login')
 */
public function login() {
    return (new RequestRedirector('profile'));
}
```

What will happen here is user is first registered. Once user registered we will forward request to login route so that user logged into account. Once logged in, we will redirect user to their profile page.

Redirect changes url in browser while forward does not, forward happens in one request only.

Every time request is forwarded it creates new stage in request. Stage number starts from 1, increased by 1 for each forward. We can get stage detail of request using `Processor::getStageDetail('detailName')`

###### Set request type

By default request is forwarded with GET HTTP method and it can changed to valid request method by passing it in second argument.

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
            ->withGetParameter(['param1'=>'value1','param2'=>'value2'])
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

*   **Ascent**: Prevent request to be forwarded to route.
*   **Descent**: Prevent request to forwarded from route.

Below code will prevents any request to be forwarded to this route.

```php
 /**
     * @Route(path='routePath')
     * @Forwarder(ascent=false)
     */
    public function methodName() {

    }
```
Attempt to forward request to this route results in exception. Just like ascent we can also prevent request to be forwarded from route.

```php
 /**
     * @Route(path='routePath')
     * @Forwarder(descent=false)
     */
    public function methodName() {

    }
```
##### Prevent incoming request / Create Private route

We can make route private so that it can used for internal only. By setting parameter `incoming = false` of `@Route` annotation will prevent any incoming request. This is useful when we to create route so that it can used for forward request only and this can also be used in CRON job.

```php
    /**
     * @Route(path='routePath',incoming=false)
     */
    public function methodName() {

    }
```

##### Required GET/POST parameter
Route might need some GET/POST parameter, Nishchay has annotation which verifies if required parameter exist. If required parameter does not found it takes action based on annotation definition.
```php
/**
 * @RequiredGet(parameter=['param1'])
 * @Route(path='routePath')
 */
public function methodName() {

}
```
Calling route path without GET parameter `param2` will result in _Route not found_ exception. We can specify any number of required parameter for route.

`@RequiredGet` is for requiring GET parameter and `@RequiredPost` annotation is for requiring POST parameter. 

We can also define both of these annotations on controller class and it will be applied to all routes within controller.

##### Only GET/POST parameter
Only GET/POST parameter means route will require exact parameter. There must not be any parameter missing from parameter list and also there must not be extra parameter.
```php
/**
 * @OnlyGet(parameter=['a','b'])
 * @Route(path='routePath')
 */
public function methodName() {

}
```
Above route will be called only if it is requested with exact parameter mentioned in `@OnlyGet` annotation. Below is list of request which will be valid or not.

| Request    | Result        |
| ----------- | ----------- |
| routePath?a=1&b=2 | Valid |
| routePath |  Fail as there are not parameter |
| routePath?a=1 |  Fails because parameter `b` is missing |
| routePath?a=1&b=2&c=3 |  Fails because parameter `c` is extra parameter |

We can also define same for POST parameter using `@RequiredPost` annotation which have parameter as `@RequiredGet` annotation. These annotation can also be defined on controller to apply to all routes within controller.

##### Scope
Defining single or multiple scope helps in events, sessions and exception handling. Using `@NamedScope` we can define single or multiple scope for the route.

```php
/**
* @Namedscope(name=secure)
* @Route(see=true)
*/
public function profile() {

}
```

Above code will create named scope `secure` for route `profile`. We can also define multiple scope for route.

When we define multiple scope for route then by default first scope in list becomes default scope of route. We can also specify default scope for the route with the help of `default` parameter of `@NamedScope` annotation.

To understand when route scope is useful, see below code.

```php
/**
* @Namedscope(name=secure)
* @Route(see=true)
*/
public function profile() {
    # To view user profile
}

/**
* @Namedscope(name=secure)
* @Route(path='profile/setting')
*/
public function profileSetting() {
    # For profile settings
}

/**
* @Namedscope(name=secure)
* @Route(see=true)
*/
public function messages() {
    # To view messages

```

Now we have multiple route for the profile and its related activity, we also to make sure that user without login must not access above defined route.

To do this we can define event for `secure` scope within which can check if user is logged or not. If user is not logged we can redirect user to login page. Please learn more about [events](/learningCenter/events/events).

Scope are also used following features

1. [Session](/learningCenter/session), Learn how to define session for scope
2. [Exception handling](/learningCenter/exceptionHandling), Learn how to define exception for scope

##### Events
Events can be defined on controller or method. Events defined on controller will be executed for each routes of controller class. These events can be executed before or after execution. There are two annotations `@BeforeEvent` and `@AfterEvent`, both of these can defined on controller or method.

Event annotation supports following parameter:

| Parameter    | Description        |
| ----------- | ----------- |
| callback | Callback method name. This can be only method if callback method is within same controller or specify with `{class}::{method}` format |
| order | Order of execution.  By default events are executed in [global, context, scope] order. This order can be changed using this parameter|
| once | Whether to execute event only once during request. If the route is forwarded then event may execute multiple time. This can be prevent if we set this parameter to true. |

```php
/**
* @BeforeEvent(callback='checkLogin')
* @Route(see=true)
*/
public function profile() {
    # View profile
}
```

###### Event response
Event must return `TRUE` to denote its success. If event is success then next action is taken. If there are no further events then route will be executed otherwise next event will be executed.

When event responds with `FALSE`, request will result in an exception `BadRequestException`. If event respond with other than `boolean` type, it will taken as request response. We can return view name or we can also redirect request. We can also forward request from event.

Even if route returns view or we redirect request, after event is executed. On failure of event does not execute after event.

##### Stages of request
Upon each request, stages starts from number 1 which keep increases by 1 count for each forward request. Each stage store details related to current request for the stage.

###### Get request/stage detail

We can get following detail of stage using `Processor::getStageDetail` method. `Processor` is facade class, in use section it should be included with `use Processor` statement. It changes on each request  forward.
| Request    | Result        |
| ----------- | ----------- |
| urlString | path of request. Empty when accessed only domain name. |
| urlParts | Url string divided by `/` |
| mode | Stage mode. Initial is always incoming, it changes to `forward` if request if forwarded. For maintenance mode its changed to `maintenance`. |
| stage | Stage number of request. Increased by 1 for each forward. |
| object | Instance of `@Route` annotation of current request. |
| priority | Priority value of current route. |
| urlex | Route path with regular expression. When we define route with dynamic segment it will be converted regex. |
| segment | List of special segment in an array. |
| context | Context name |
| scopeName | Scope name of route. |