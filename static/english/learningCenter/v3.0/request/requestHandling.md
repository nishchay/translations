#### Request Handling

##### Introduction

A request is handled by a **route method**, a method of a **controller** class. Nishchay matches the URL and HTTP method of the request to a route, runs the events around it, calls the method and sends back what it returns.

A class with the `Controller` attribute is a controller.

```php
use Nishchay\Attributes\Controller\Controller;

#[Controller]
class NamasteController
{

}
```

##### Route

A controller method is a route method when all of the following are true.

1. It has the `Nishchay\Attributes\Controller\Method\Route` attribute.
2. It is declared in the controller class itself. Methods inherited from a parent class are ignored.
3. It is public.
4. It is not static.
5. Its name does not start with an underscore.

###### Simple route

```php
namespace Application\Controllers\Direct;

use Nishchay\Attributes\Controller\Controller;
use Nishchay\Attributes\Controller\Method\Route;

#[Controller]
class NamasteController
{

    #[Route(path: 'namaste')]
    public function namaste()
    {
        return 'namaste';
    }

}
```

This creates the route _namaste_, reachable at `{HOST}/namaste`. If the application is hosted at `http://app.nishchay.local`, the address is `http://app.nishchay.local/namaste`.

**What the route returns.** The default response type is _view_, so a returned string is the **name of a view** to render, not text to print. In the example above, Nishchay looks for a view called _namaste_. See [response handling](/learningCenter/request/responseHandling) to return JSON, XML or raw text instead.

###### Landing route

The landing route is the one that runs when the visitor opens only the domain name. Set it with the `landing` key in `settings/configuration/routes.php`.

```php
return [
    'landing' => 'namaste',
    ...
];
```

###### Multiple segments

A path can be any string. Parts separated by `/` are called segments.

```php
#[Route(path: 'user/messages')]
public function messageList()
{
    ...
}
```

This path has two segments, _user_ and _messages_. A leading or trailing `/` is ignored, and the path cannot be empty.

###### Method name as route path

Pass `true` as the path and the method name becomes the path.

```php
#[Route(path: true)]
public function namaste()
{
    ...
}
```

The route is _namaste_.

##### Placeholder segment

A segment written in curly braces is a **placeholder**. It matches any value of the type you declare for it. Declare the types with the `Nishchay\Attributes\Controller\Method\Placeholder` attribute.

```php
use Nishchay\Attributes\Controller\Method\Route;
use Nishchay\Attributes\Controller\Method\Placeholder;

#[Route(path: 'user/messages/{userId}')]
#[Placeholder(['userId' => 'int'])]
public function messageView($userId)
{
    ...
}
```

The key of the array is the segment name and the value is its type. Every placeholder in the path needs an entry, and an entry for a name that is not in the path is an error.

###### Placeholder types

| Type | Matches |
|---|---|
| `int`, `number` | Digits only |
| `alphanum` | Letters and digits |
| `string` | Letters, digits, dot, underscore and hyphen |
| `bool`, `boolean` | `0` or `1` |
| array of values | Only the listed values |

Use an array when only a few values are valid.

```php
#[Route(path: 'color/{name}')]
#[Placeholder(['name' => ['red', 'blue']])]
public function color($name)
{
    ...
}
```

`color/red` matches. `color/green` does not match any route, so the response is a 404.

You can use as many placeholders as you need.

```php
#[Route(path: 'blog/{year}/{slug}')]
#[Placeholder(['year' => 'int', 'slug' => 'string'])]
public function post($year, $slug)
{
    ...
}
```

###### Optional segment

Put `?` before the placeholder to make it optional.

```php
#[Route(path: 'profile/?{userId}')]
#[Placeholder(['userId' => 'int'])]
public function userProfile($userId = null)
{
    ...
}
```

Both `profile` and `profile/12` reach this method. Give the parameter a default value, because it has no value when the segment is missing.

Placeholder values arrive as strings, even for `int`. Cast them yourself if you need a number.

##### Receiving request data in route parameters

Parameters of the route method are filled for you. You do not read them from the request by hand.

```php
#[Route(path: 'order/{orderId}')]
#[Placeholder(['orderId' => 'int'])]
public function order($orderId, ?string $note, array $GET, array $POST, $coupon = null)
{
    ...
}
```

* A parameter with the same name as a placeholder gets the segment.
* Any other name is looked for in this order: uploaded files, POST, GET. The first match wins, so a POST value hides a GET value of the same name.
* `$GET` and `$POST` (in capitals) get all GET and all POST parameters as arrays.
* A parameter with a default value uses it when nothing is found. A nullable parameter such as `?string $note` gets `null`.
* A parameter that is not optional and cannot be found throws `Not able to find what to assign to [name] parameter`.
* A parameter with a class as its type hint is created by [dependency injection](/learningCenter/dependencyInjection).

Read more about [request parameters](/learningCenter/request/requestParameter).

##### Prefix for all routes of a controller

The `Nishchay\Attributes\Controller\Routing` attribute adds a prefix to every route of the controller.

```php
use Nishchay\Attributes\Controller\Controller;
use Nishchay\Attributes\Controller\Routing;

#[Controller]
#[Routing(prefix: 'user')]
class User
{

    #[Route(path: true)]
    public function profile()
    {
        ...
    }
}
```

The route is _user/profile_.

###### Skip the prefix for one route

Pass `prefix: false` to `Route` to keep the path exactly as written.

```php
#[Route(path: 'login', prefix: false)]
public function login()
{
    ...
}
```

This route is _login_, not _user/login_.

###### Prefix from the class name

`prefix: 'this.base'` uses the class name without its namespace.

```php
#[Controller]
#[Routing(prefix: 'this.base')]
class User
{

    #[Route(path: true)]
    public function profile()
    {
        ...
    }
}
```

The route is _User/profile_. The `case` parameter of `Routing` changes the case of the prefix.

| Value | Prefix for class `User` |
|---|---|
| `lower` | `user` |
| `upper` | `USER` |
| `camel` | `user` (first character in lower case, the rest unchanged) |

```php
#[Routing(prefix: 'this.base', case: 'lower')]
```

`Routing` also takes a `pattern` parameter that generates routes from method names. Read more about [route patterns](/learningCenter/request/routePattern).

##### Route HTTP methods

By default a route answers every HTTP method. Use the `type` parameter of `Route` to limit it to one method or to a list. The names are not case sensitive.

```php
#[Route(path: 'profile', type: 'GET')]
public function profile()
{
    ...
}

#[Route(path: 'profile', type: ['POST', 'PUT'])]
public function saveProfile()
{
    ...
}
```

Supported methods are `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD`, `TRACE` and `CONNECT`. A request with any other method for that path gets a 404.

The same path can be used again for another HTTP method, as above. The combination of **path and HTTP method must be unique** in the application. A route without `type` already owns every method, so this fails when the application starts with `Route [profile] conflicted.`

```php
#[Route(path: 'profile')]
public function profile() { ... }

#[Route(path: 'profile', type: 'POST')]
public function saveProfile() { ... }
```

##### Route priority

Routes are checked in the order they are stored. A route with a placeholder can therefore hide a fixed route stored after it.

```php
#[Route(path: 'post/{name}', type: 'GET')]
#[Placeholder(['name' => 'string'])]
public function viewPost() { ... }

#[Route(path: 'post/create', type: 'GET')]
public function createPost() { ... }
```

`post/create` is also a valid `{name}`, so `viewPost` runs for it and `createPost` is never reached.

The `Nishchay\Attributes\Controller\Method\Priority` attribute fixes this. The route with the **higher value is checked first**. The default is 100.

```php
use Nishchay\Attributes\Controller\Method\Priority;

#[Priority(value: 200)]
#[Route(path: 'post/create', type: 'GET')]
public function createPost() { ... }

#[Route(path: 'post/{name}', type: 'GET')]
#[Placeholder(['name' => 'string'])]
public function viewPost() { ... }
```

Now `post/create` goes to `createPost`, and every other `post/...` goes to `viewPost`.

##### Stage route

An application runs in a stage: _local_, _test_ or _live_. It is set in `settings/configuration/application.php`. A route can be limited to the _local_ and _test_ stages with the `stage` parameter. It is never available in _live_.

```php
#[Route(path: 'checkTest', stage: 'test')]
public function testRoute()
{
    ...
}
```

This route exists in the _test_ stage only. In any other stage the URL returns 404. Use an array for both stages.

```php
#[Route(path: 'debug', stage: ['local', 'test'])]
```

Any other stage name is an error.

##### Abstract (static) route

Pages like _About us_, _Terms_ or _Help_ only render a view. For them you can declare the route on an **interface** method. No controller instance is created and no code runs. Nishchay renders the view directly.

```php
#[Controller]
interface StaticController
{
    #[Route(path: 'help')]
    public function help();
}
```

The view is chosen in one of two ways.

* **Route path.** With no other setting the path is the view name, here _help_. The `abstractViewPath` setting in `settings/configuration/response.php` is put in front of it. With `'abstractViewPath' => 'static'` the view is `static/help`.
* **Response attribute.** Use the `view` parameter to name the view yourself.

```php
use Nishchay\Attributes\Controller\Method\Response;

#[Route(path: 'help')]
#[Response(type: 'view', view: 'static/help')]
public function help();
```

An interface is the cleanest choice, because a class with abstract methods must also be declared abstract.

##### Redirect request

To redirect, return a `RequestRedirector` from the route. Create it with one of these helpers of the `Request` class.

| Helper | Use |
|---|---|
| `Request::getRedirectWithin($route)` | Redirect to a route of this application |
| `Request::getRedirect($url)` | Redirect to a full URL, in or outside the application |

```php
use Nishchay\Http\Request\Request;

#[Route(path: 'admin')]
public function admin()
{
    if ($this->isUserAdmin() === false) {
        return Request::getRedirectWithin('unauthorized');
    }
    ...
}
```

`getRedirectWithin` adds the base URL of the application, so `unauthorized` becomes `http://app.nishchay.local/unauthorized`.

To redirect outside the application, check the address first. Never redirect to a URL taken from the request without checking it.

```php
#[Route(path: 'goto')]
public function goTo()
{
    $url = Request::get('to');
    if (UrlChecker::isUrlSafe($url)) {
        return Request::getRedirect($url);
    }

    return Request::getRedirectWithin('unsafeUrl');
}
```

`UrlChecker` is your own class.

**NOTE** `new RequestRedirector('path')` sends its argument as the `Location` header **unchanged**. A relative value is resolved by the browser against the current URL, and can end up as `/user/user/profile`. Use `getRedirectWithin` for routes of your application.

A redirect changes the URL in the browser and always makes a new `GET` request.

##### Forward request

A forward sends the request to another route **inside the same request**. The browser does not know about it and the URL does not change.

```php
use Nishchay\Http\Request\RequestForwarder;

#[Route(path: 'register')]
public function register()
{
    # save the user, then log in
    return new RequestForwarder('login');
}

#[Route(path: 'login')]
public function login()
{
    # log the user in, then show the profile
    return Request::getRedirectWithin('profile');
}
```

Here the user registers, the request is forwarded to _login_, and _login_ redirects the browser to the profile.

|  | Redirect | Forward |
|---|---|---|
| Browser URL | Changes | Stays |
| Requests | New request | Same request |
| HTTP method | `GET` | Any |
| Parameters | Query string only | GET and POST values you pass |

Every forward starts a new **stage** of the request. See _Stages of request_ at the end of this page.

###### HTTP method

The default method is `GET`. Pass another one as the second argument.

```php
new RequestForwarder('routePath', 'POST')
```

###### GET parameters

```php
(new RequestForwarder('routePath'))
    ->withGetParameter(['param1' => 'value1', 'param2' => 'value2']);
```

###### POST parameters

```php
(new RequestForwarder('routePath', 'POST'))
    ->withPostParameter(['param1' => 'value1', 'param2' => 'value2']);
```

POST parameters are ignored when the method is not `POST`.

In the target route, read them as usual, for example `Request::post('param1')`.

###### Clear the request store

Data added to the [request store](/learningCenter/request/requestStore) is still there in the next stage. Clear it while forwarding with `withFlushRequestStore`.

```php
(new RequestForwarder('routePath'))->withFlushRequestStore();
```

###### Prevent forwarding

Two parameters of the `Nishchay\Attributes\Controller\Method\Forwarder` attribute control what is allowed.

* `ascent: false`: no request can be forwarded **to** this route.
* `descent: false`: this route cannot forward a request **to another route**.

```php
use Nishchay\Attributes\Controller\Method\Forwarder;

#[Route(path: 'routePath')]
#[Forwarder(ascent: false)]
public function methodName() { ... }

#[Route(path: 'other')]
#[Forwarder(descent: false)]
public function other() { ... }
```

Breaking a rule throws a `NotSupportedException`, such as `Ascent forward not allowed on route [routePath].`

##### Private route

Set `incoming: false` to refuse every direct request. The route can still be reached by a forward or by the command line, so it suits internal routes and cron jobs.

```php
#[Route(path: 'routePath', incoming: false)]
public function methodName()
{
    ...
}
```

A direct request to this route gets a 404.

##### Required parameters

`RequiredGet` and `RequiredPost` list the GET or POST parameters that must be sent. If one is missing, the request ends with `BadRequestException: Request could not be satisfied.` (400).

```php
use Nishchay\Attributes\Controller\RequiredGet;
use Nishchay\Attributes\Controller\RequiredPost;

#[Route(path: 'search')]
#[RequiredGet(parameter: ['q', 'page'])]
public function search() { ... }

#[Route(path: 'save', type: 'POST')]
#[RequiredPost(parameter: ['name'])]
public function save() { ... }
```

`search?q=a` fails because `page` is missing, and `search?q=a&page=1` works. Extra parameters are allowed.

Both attributes can also be put on the controller class. They then apply to every route of the controller.

##### Only these parameters

`OnlyGet` and `OnlyPost` require **exactly** the listed parameters. A missing one and an extra one are both rejected.

```php
use Nishchay\Attributes\Controller\OnlyGet;

#[Route(path: 'routePath')]
#[OnlyGet(parameter: ['a', 'b'])]
public function methodName() { ... }
```

| Request | Result |
|---|---|
| `routePath?a=1&b=2` | Works |
| `routePath` | Fails, parameters are missing |
| `routePath?a=1` | Fails, `b` is missing |
| `routePath?a=1&b=2&c=3` | Fails, `c` is extra |

Failures are the same 400 as for required parameters. `OnlyPost` works the same for POST parameters, and both can be put on the controller class.

##### Scope

A scope is a name you give to a group of routes. Events, sessions and exception handlers can then be attached to the name instead of to each route. Define it with the `Nishchay\Attributes\Controller\Method\NamedScope` attribute.

```php
use Nishchay\Attributes\Controller\Method\NamedScope;

#[Route(path: true)]
#[NamedScope(name: 'secure')]
public function profile() { ... }

#[Route(path: 'profile/setting')]
#[NamedScope(name: 'secure')]
public function profileSetting() { ... }

#[Route(path: true)]
#[NamedScope(name: 'secure')]
public function messages() { ... }
```

These three routes share the _secure_ scope. Define one event for _secure_ that checks the login and all three are protected. Read more about [events](/learningCenter/events/events).

A route can have several scopes. Pass an array in `name`. The first one is the default scope, and the `default` parameter chooses another.

```php
#[NamedScope(name: ['secure', 'admin'], default: 'admin')]
```

Scope is also used by

1. [Session](/learningCenter/session)
2. [Exception handling](/learningCenter/exceptionHandling)

##### Events

An event is a method that runs **before** or **after** a route. Declare it on a route method with the attribute `Nishchay\Attributes\Controller\Event\BeforeEvent` or `Nishchay\Attributes\Controller\Event\AfterEvent`.

```php
use Nishchay\Attributes\Controller\Event\BeforeEvent;

#[Route(path: true)]
#[BeforeEvent(callback: 'checkLogin')]
public function profile()
{
    ...
}

public function checkLogin()
{
    return isset($_SESSION['user']);
}
```

Parameters of both attributes:

| Parameter | Description |
|---|---|
| callback | Method name when the method is in the same controller, or `{class}::{method}` for a method of another class |
| order | Order of execution. Events run in the order global, context, scope. Pass an array to change it |
| once | Run the event only once per request. A forward can run a route several times, so use `true` to avoid repeats |

Events can also be attached to scopes, contexts or the whole application. See [events](/learningCenter/events/events).

###### What an event returns

| Return value | Result |
|---|---|
| `true` | Success. The next event runs, and then the route |
| `false` | The request ends with `BadRequestException` (400) |
| Anything else | It becomes the response of the request. The route is **not** called |

An event can return a view name, a redirect or a forward in the same way a route does.

An after event always runs, even when a before event has already replaced the response. It does not run when a before event returns `false`.

##### Stages of request

A request starts at stage 1. Each forward adds one stage. Every stage keeps its own details, read with `Processor::getStageDetail`.

`Processor` is a facade, so add `use Processor;` at the top of the file.

```php
use Processor;

$mode = Processor::getStageDetail('mode');
$all = Processor::getStageDetail();   # every detail of the current stage as an array
```

An unknown name throws `Invalid state detail`.

| Detail | Description |
|---|---|
| urlString | Path of the request. Empty when only the domain was opened |
| urlParts | Path split by `/` |
| mode | `incoming` for the first stage, `forwarded` after a forward. `maintenance` when the application is in maintenance mode |
| stage | Stage number. Starts at 1 and grows by 1 for each forward |
| object | The `Route` attribute of the current route |
| priority | Priority of the current route |
| urlex | Route path as a regular expression. A route with placeholders is turned into one |
| segment | List of placeholder segments of the route |
| context | Name of the context of the route |
| scopeName | Scope name of the route, or `false` |

```php
#[Route(path: 'register')]
public function register()
{
    return new RequestForwarder('login');
}

#[Route(path: 'login', incoming: false)]
public function login()
{
    # Processor::getStageDetail('mode')  => 'forwarded'
    # Processor::getStageDetail('stage') => 2
}
```
