#### Prototype

Prototype is already implemented task which can be configured to your code. To use prototype we just need provide it entity and form class. Apart from this it also provides various other configuration and callback method which executes before or after certain actions. This configuration and callback differs for each prototype. Nishchay provides following prototypes:

1. Auth
2. Register
3. CRUD

Prototype must be used with entity. It does not connect directly to table, it must be through entity only.

##### Auth : Account

This prototype allows authorizing(login) user within an application. This prototype is part of `AccountPrototype`. Below code demonstrates how it can be used.

```php
use Nishchay\Prototype\AccountPrototype;
use Application\Entities\User;

class AccountController{

    /**
    * @Route(path='account/auth',type=POST)
    */
    public function auth(AccountPrototype $account) {
        $response = $account->getAuth(User::class)
                    ->execute();
    }
}
```

This will authenticate user using `User` entity as provided in `getAuth` method. When we call `execute` method of prototype, user will be authenticated by following steps when request is post:

1. Prototype will find user by matching email. Entity's email property with email received in POST(email parameter) request.
2. If user is not found it will throw `BadRequestException` with `User does not exists.` message.
3. If returned record has `isActive` property,
   1. It must be `true`.
   2. If it's `false` then `BadRequestException` exception is thrown with `User is not active.` message.
4. If returned record has `isUserVerified` property,
   1. It must be `true`.
   2. If it's `false` then `BadRequestException` exception is thrown with `User is not verified.` message.
5. Password will be verified PHP's `password_verify` function.
6. AccessToken(OAuth2) will be generated.
7. It will respond with instance of ` Nishchay\Prototype\Account\Response\AccountResponse` class (Read more about this in _Account Response_ section).

###### Account : email and password field

By default prototype will find email from `email` parameter and password form `password` request parameter. Property should exists with same name in `Entity` class.

If you are authenticating user with its username, use `setEmailName` and set email field as `username`.

```php
$account->setEmailName('username');
```

If you have password field with other name, it can be changed to name you have.

```php
$account->setPasswordName('userPassword');
```

**NOTE** Field must exists with same name in entity.

###### Validation

If you want to validate before authorizing an user, use `setForm` method.

```php
$account->getAuth(User::class)
        ->setForm(AuthForm::class)
        ->execute()
```

When we set form then form will be validated first. If there's validation error then it will be returned.

When we use form in auth prototype then there must `getEmail` method should be used for `email or username` field and `getPassword` method for password field.

###### Password verification

By default password is verified with PHP's `password_verify` function. If password is not stored using PHP`s `password_hash` function, we can set callback method which will be called to verify password. This callback method must verify password.

```php
$account->verifyPassword(function($requestPassword, $userPassword) {
    return md5($requestPassword) === $userPassword;
});
```

Here first parameter is password recieved in request and second parameter password from database.

###### Post authentication

Once user is authenticated, we can also do more verification by setting callback method which will be called after user is authenticated. In this callback we can implemented more verification apart from password verification.

```php
$account->postAuth(function($user) {
    # Implement more verification
})
```

##### Session / OAuth2
When user is authenticated or registered, Account prototype creates OAuth2 token by default. If we don't want OAuth2 token after success, we can disable it.

```php
$account->generateOAuth2(false);
```

Above will disable OAuth2 to be generated after success.

###### Session
Account prototype can also save data to session upon. This is disabled by default. To enable:

```php
$account->setSession(true);
```

This will enable session for the account prototype, below data will set into session:

1. isLogged (Set to true)
2. userId (User id of authenticated or registered user)


##### Register : Account
This prototype is for registering an user within application.

```php

use Application\Forms\Register;
use Application\Entities\User;

/**
* Creates an account of user.
* 
* @Route(path='user',type=POST)
*/
public function register(AccountPrototype $accountPrototype)
{
    $response = $accountPrototype->getRegister(User::class)
            ->setForm(Register::class)
            ->execute();
}
```

This will validates requests, if there's an error it returns error. Upon successful validation, form data inserted to  database using provided entity.
By default upon success OAuth2 is created, this can be disabled or can be configured to set session data(See _Session / OAuth2_ section).

###### Ignore fields
In register form, there can be fields which should not be inserted to database. We can omit those fields by using `setIgnoreFileds`. This method accepts list of fields need to ignored in array.

```php
$account->setIgnoreFileds(['isTermAccepted']);
```

###### Before register
If we need to do some task before registration, that can be setting callback method which will be called just before form data saved.

```php
$account->preRegister(function($form) {
    # Implement pre register task
})
```

###### After register
Just like pre register, we can also have post register callback which will be called after registration.


```php
$account->postRegister(function($form) {
    # Implement post register task
})
```

##### Account Response

`Auth` and `Register` prototype both returns instance of `Nishchay\Prototype\Account\Response\AccountResponse` which can have following details(On success):

1. User detail (User record)
2. success (`true` for success)
3. OAuth2 token (If enabled)
4. CSRF (If CSRF has been in form which was used with prototype)
5. Validation errors

To get above details use below methods:

|Method|Description|
| - | - |
| getUserDetail | Returns instance of entity for the user record. |
| getAccessToken | Returns OAuth2 token |
| isSuccess | Returns `true` on success |
| getErrors | Returns validation errors |
| getCsrf | Returns CSRF token if CSRF has been set in form |

##### CRUD

This prototype contains set of methods to implement CRUD operation in an application. This CRUD includes:

1. Fetch list of records (With pagination)
2. Fetch single record
3. Insert record
4. Update record
5. Remove record

Create instance of CRUD prototype:

```php
$crud = new Nishchay\Prototype\Crud(Entity::class);
```

This prototype can also be created using console command. When we create prototype using console command its creates controller along with entity and form class.

###### Fetch list of records
Using `get` method we ca get list of records of provided entity. This also comes with pagination,  pagination can be disabled.

```php
use Nishchay\Prototype\Crud;

/**
* @Route(path=items,type=GET)
* @Response(type=json)
*/
public function getItems() {
    $crud = new Crud(Entity::class);
    return $crud->get();
}
```

Pagination works on `limit` and `offset` GET parameter.

###### Pagination limit
When CRUD get prototype is used, we ca get number of records based on `limit` get parameter. As one can request any number of requests, we can specify min and max limit.

If we have set min limit to 10 and we try to fetch record using limit=4 then prototype will return 10 records. If we set max limit to 50 records and one request with limit=100 then prototype returns 50 records only. This is like if one request with lower limit then we set, it resets to min limit, same applies to max limit.

To set min limit:

```php
$crud->setMinLimit(10);
```

To set max limit;

```php
$crud->setMaxLimit(50);
```

By default min limit is set to 10 and max set to 50.

###### Disable pagination

To disable pagination call `pagination` method with `false` parameter.

```php
$crud->pagination(false);
```

###### Fetch single record

To fetch single record use `getOne` method, this method accepts one argument which should be value of identity property.

```php
use Nishchay\Prototype\Crud;

/**
* @Route(path='item/{itemId}',type=GET)
* @Placeholder(itemId=int)
* @Response(type=json)
*/
public function getItem(int $itemId) {
    $crud = new Crud(Entity::class);
    return $crud->getOne($itemId);
}
```

When record not found, it throws `BadRequestException` exception. To respond with our response rather than `BadRequestException` exception, use `setOnFailure` which accepts callback method.

When callback method returns `false` prototype will throw `BadRequestException` exception. When method returns with value other than `false`, it can be used as route response.

###### Insert record

To insert record using this prototype we only have to set form class and then call `insert` method.

```php
use Nishchay\Prototype\Crud;

/**
* @Route(path='item',type=POST)
* @Response(type=json)
*/
public function insert() {
    $crud = new Crud(Entity::class);
    return $crud->setForm(Form::class)
                ->insert();
}
```

###### Update record
To update record using this prototype we only have to set form class and then call `update` method with value of identity property.

```php
use Nishchay\Prototype\Crud;

/**
* @Route(path='item/{itemId}',type=PUT)
* @Placeholder(itemId=int)
* @Response(type=json)
*/
public function update(int $itemId) {
    $crud = new Crud(Entity::class);
    return $crud->setForm(Form::class)
                ->update($itemId);
}
```

###### Remove record

To remove record, we only ave to call `update` with value of identity property. 

```php
use Nishchay\Prototype\Crud;

/**
* @Route(path='item/{itemId}',type=DELETE)
* @Placeholder(itemId=int)
* @Response(type=json)
*/
public function remove(int $itemId) {
    $crud = new Crud(Entity::class);
    return $crud->remove($itemId);
}
```

###### CRUD Response
All methods `insert`, `update` and `remove` returns array with only key `message` which has success or failure message. If want to change this response read CRUD events.

###### CRUD events

All methods `insert`, `update` and `remove` has similar events which are listed below:

1. Before save(insert, update or remove)
2. Record not found (calls failure callback if set)
3. Success (calls success callback if set)
4. Failure (calls failure callback if set). When record could not be saved.

To set before save event:

```php
$crud->setBefore(function($entity) {

})
```

Response of above method must be `true` to indicate success of event. When this returns `false`, prototype throw exception. In this case if there's failure callback set then it will be called.

To set failure callback: This will be called when record is not found or record could not be saved.

```php
$crud->setOnFailure(function($code) {

});
```

To set success callback: This will be called when record is inserted, updated or removed.

```php
$crud->setOnSuccess(function($code) {

});
```

Both of this callback receives code which are listed below: (This code depends on prototype used)

1. TERMINATED_BEFORE_SAVE
2. FAILED_TO_INSERT
3. RECORD_REMAINED_SAME
4. RECORD_NOT_FOUND
5. RECORD_NOT_REMOVED

###### Event response

When event callback respond with other than boolean then it will be considered as response of route.