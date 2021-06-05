#### Session

Nishchay session management is powerful and easy to use. It can be used multiple ways, as an example scope session which are available to processing route's scope only and it also has number of type of save handler like file, DB and Cache.

Session are started when its first usage is made. We don't have to use any of php function or variable to manage session instead Nishchay provides more convenient way to use it. `As Nishchay has its own implementation for managing session, please do not use any of php function or variable($_SESSSION).`

##### Managing session data

Writing, reading or removing session are same for each type of session. `Nishchay\Session` namespace contains all types of session. Creating instance of any session type will allow us to read or maintain session data.

Below example demonstrate how instance of normal session is created.

```php
$session = new Nishchay\Session\Session()
```

Now as we have instance of session class we can write session data as shown below.

```php
$session->userId = 12345
```

As per above `userId` will be added to session. Once we add data to session it can be accessible form anywhere, we just need to create instance session class then access it.

###### Reading session data

```php
echo $session->userId
```

Session data can also be read or write as an array way.

```php
$session['userId'] = 12345;
echo $session['userId'];
```

We can create any number instance of session class. Session data write to one class, also available to other instance.

```php
$session1 = new Nishchay\Session\Session();
$session2 = new Nishchay\Session\Session();
$session1->userId = 12345;

# Prints 12345
echo $session1->userId;

# Prints 12345, here userId will also be available $session2 instance.
echo $session2->userId;
```

Session data are shared among each instance of class, changing on one place gets reflected to other place also.

###### Iterate session data

Session data can be iterated just as an array.

```php
foreach ($session as $key => $value) {
    echo "{$key} : {$value}"
}
```

###### Remove session data

Session data are removed by using `unset` method.

```php
unset($session->userId)
```

It can also be removed as array way

```php
unset($session['userId'])
```

##### Types of session

Each session type are differentiated by its availability and its lifetime but usage of each types of session remains same.

###### Normal session

Simplest form of session, which works the same as php core session. It doesn't have any special feature associated with it. Session of this type created using `Nishchay/Session/Session` class.

###### Scope

As we know route can have named scope defined using `@NamedScope` annotation on route method. When we add data to scope session it will be available to all routes which belongs to same scope.

When we create instance of Scope session it first checks route's scope, and then it fetches all data of that scope. Data added to one scope won't be available to other scope.

```php

#[Route(path: 'profile')]
#[NamedScope(name: 'secure')]
public function profile() {
    // This create session for scope 'secure' only.
   $session = new \Nishchay\Session\Scope();

    // This session data gets added to 'secure' scope.
   $session->code = 12345;
}

#[Route(path: 'profile')]
#[NamedScope(name: 'direct')]
public function helpProfile() {
   // This create session for scope 'direct' only.
   $session = new \Nishchay\Session\Scope();

   $session->code = 54321;
}
```

Here we have added `code` to `secure` and `direct` scope session. Both of these session have their separate copy. Any route which belongs to `secure` will have `code = 12345` while routes belongs to `direct` scope will have `code = 54321`.

**What if route does not have scope.**

In this case, session won't be created and it will through exception with message _'Route does not have scope. You should define scope for route to use scope session.'_.

###### Route with multiple scope

When route defines multiple scope, there always default scope for the route. Creating Scope session without passing scope name will use default scope as defined in `@NamedScope`.

```php

#[Route(path: 'profile')]
#[NamedScope(name: ['scope1', 'scope2', 'scope2'])]
public function profile() {
    $session = new \Nischchay\Session\Scope('scope2');
}
```

Now any data added to `$session` will be added for scope `scope2`.

###### Context

All controller belongs to context and we can create session which are available to that context. `Context` session class allows us to create session data to be available for controller's context only. On creating instance of `Context` session class, it fetches data of that context by checking route's context. Context session can be created for any route as all routes are belongs context.

```php
namespace Application/Access/Secure/Controllers;
class SecureRoutes {

    #[Route(path: 'profile')]
    public function profile() {
       $session = new \Nishchay\Session\Context();
       $session->code = 12345;
    }
}
```

Above will add session data named 'code' to context 'Application/Access/Secure'.

###### Read limit / Number of time read

This type session data available till it reaches its read limit. Suppose we have session data 'token' to be accessed for at most 3 times. Once we read 'token' data 3 times, it get removed and then next time it won't be available.

```php
$session = new \Nishchay\Session\ReadLimit(3);
$session->token = 'TK12345';

// Reading for the first time.
echo $session->token;

// Reading second time.
echo $session->token;

// Reading third time, now this time as its limit has been reached Nishchay removes this data and returns it.
echo $session->token;

// This will gives error as it was removed when it was read 3rd time.
echo $session->token;
```

Instance created with read limit will applies to all data added to it. This read limit cannot be changed later. If we want another read limit then we have to create instance with another limit.

Setting data again resets its read count to zero. Suppose data has read limit 5 and we have read it for 3 time and updating this data will set read count zero. Then again reading it increments from zero.

###### Till next request count

This type of session allow us to create session data which are available for next number of request. Below code demonstrates how to create session data which are available for next 5 number of request.

```php
$session = new \Nishchay\Session\TillNextRequestCount(5);
$session->accessToken  = 'ASDF1234';
```

_accessToken_ session data will be available for next 5 request and then it will be removed.

Updating session data will renew its lifetime for next 5 request again. Just like `ReadLimit` session, count of next request cannot be changed. If we want another limit for session, new instance should be created with new limit.

###### Flash Session

Flash session are session which has lifetime to next request only. Many library or framework provide flash session, but Nishchay does not. Because it can be achieved by creating session with lifetime of next request only.

```php
$session = new \Nishchay\Session\TillNextRequestCount(1);
$session->success  = true;
```

###### Conditional

Session data can also saved and accessed based on its save and read condition. Creating instance of `Conditional` session requires two callback parameter, first being read check and second for save check. Upon adding/updating session data, Nishchay first calls its save check. If save check returns _true_, data gets added. Failing throws error. Same rule applies to read.

Read and Save check callback must be string, as it gets persisted along with session data. Callback string should be {CLASS}::{METHOD} or {METHOD} name only.

```php
$session = new \Nishchay\Session\Conditional('ConditionalCheck::readCheck', 'ConditionalCheck::saveCheck');
```

When we add session data `ConditionalCheck::saveCheck` will be called with data name and its value. If method does not return `true` then exception will be thrown. Same goes for we access session data.

##### Cookie

Like session, cookie also works same as session exception cookie value should be only string while in session we can store any kind of value.

Cookies are stored on client machine and it also provide option. These option once set, it will be applied to all future cookies using same instance. If there are two instance of `Cookie` class, they will have their separate options.

See below methods

| Name        | Description                                                                                                                                                                           |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| setExpiry   | Expiry time in seconds.                                                                                                                                                               |
| setPath     | Path where cookie need to be available. Cookie will be available to all of its sub directory of path. By default this is current, means it will be available to path where it was set |
| setDomain   | Domain on which only need to be available to                                                                                                                                          |
| setSecure   | Set to `true` to allow cookies over HTTPS only.                                                                                                                                       |
| setHttpOnly | Set to `true` to allow cookies over HTTP protocol                                                                                                                                     |

You can learn more about cookie options [here](https://www.php.net/manual/en/function.setcookie.php).

##### Session storage location

Session data can be stored in file, database or cache which can be specified in `session.storage` setting `application.php`.

###### File/Default

This is default save handler with default save path `/persisted/sessions/`. This storage path can changed by changing config `session.storagePath`. We recommened this location to be outside of application directory.

###### Database

To store session data in db change `session.storage` to `db`. For storage `db` setting `session.db` also need to be modified. See below setting database storage.

| Name       | Description                                                                                       |
| ---------- | ------------------------------------------------------------------------------------------------- |
| connection | Connection name, keeping it NULL uses default connection as provided in database setting.         |
| table      | Table name where session data will be stored, keep it NULL will consider `Session` as table name. |

Table structure for `Session`:

```sql
CREATE TABLE `Session` (
  `sessionId` VARCHAR(250) NOT NULL,
  `data` TEXT DEFAULT NULL,
  `accessAt` datetime DEFAULT NULL
) DEFAULT CHARSET=latin1
```

###### Cache

To store session data in db change `session.storage` to `cache`. For storage `cache` setting `session.cache` also need to be modified. See below setting cache storage.

| Name   | Description                                                                                                   |
| ------ | ------------------------------------------------------------------------------------------------------------- |
| name   | Cache config name, it should be null if want to use default cache config as defined in cache.php config file. |
| expiry | Cache expiry time in seconds, this should be greater than 300 seconds.                                        |
