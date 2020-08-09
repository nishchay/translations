#### CSRF (Cross Site Request Forgery)

CSRF is security vulnerability that allows an attacker to perform actions that they do not intend to perform. CSRF is to prevent unauthorized request to server. Nishchay prevents this attack with the help of CSRF token.

When web application renders form, application generates token & saves onto server then it sends along with form. When the form is submitted, token also be submitted along with request then on server nishchay checks received token with server token. If there's mismatch request is discarded.

In nishchay we don't have worry about how token is generated, how it saves in server and how it verifies. Form builder and validator class also comes with support of CSRF which we will discuss. It works same for both of these class.


##### Create instance

When we create instance of CSRF, we also need to pass name for which it need to be used. This is because CSRF token will be saved with the `csrs_{name}` name in session. So when we create CSRF instance, choose unique name.

```php
$csrf = new CSRF('test-csrf');
```

##### How token should be received

By default CSRF is configured to be received in POST request with name `csrf`. We can change how token should be received. We can change it following location

1. GET request
2. POST request
3. In Header

Below code show how this location can be changed:

```php
$csrf->setWhere(Request:GET);
```
This changes CSRF token be received in GET request

##### Change name

By default token should comes in `csrf` parameter, we can change it anything we want. 


```php
$csrf->setName('token');
```

Above code changes token to be received in `token` parameter.

If we have configured token to be received in header then change name something like `X-CSRF-Token`.

##### Fetch token

To fetch token use `getValue` method.

**NOTE** token is not created when instance of CSRF class is created but its created when first we fetch token. This is to ensure that token should not be created unless you use. If we have not created then CSRF verification still fails because there won't be token received.

```php
$csrf->getValue();
```

##### Change token length

By default 64 character length token is generated, we can change it any length we want. We recommend not to decrease its length than 64 character.

```php
$csrf->setLength(128);
```

##### Printing CSRF

If we have configured CSRF for GET or POST request then printing instance of CSRF prints csrf token with tag:

```php
$csrf = new CSRF('test-form');

echo $csrf;
```

Above will output:

```html
<input type='hidden' name='csrf' value='{TOKEN}' />
```

But if CSRF token to be set header then `echo $csrf` returns only token:

##### Verify CSRF token

We only need to call `verify` method when we receive request. If there's mismatch in token then this method throws `Nishchay\Exception\BadRequestException` exception.

```php
if ($csrf->verify()) {
    # Go for further processing.
}
```

##### Example

To understand how we can use CSRF, we will create simple demo:

###### Step: 1 Create form with CSRF token

Let's create one GET route where we will render form with CSRF token.

```php
use Nishchay\Security\CSRF;
use Nishchay\Http\Request\RequestStore;

/**
* @Route(path='csrf', type='GET')
* @Response(type='view')
*/
public function csrfGET() {
    $csrf = new CSRF('test-csrf');

    RequestStore::set('csrf', $csrf);
    RequestStore::set('action', '/csrf');

    return 'testForm';
}
```

###### Step 2: Create view

As we are returning view we need to create one view called `testForm.twig` and below code will be its content:

```html
<form action="{{action}}" method="POST">
    {{csrf}}
    <input type="submit" value="submit"/>
</form>
```

###### Step 3: Verify

Now when form is submitted, we will verify CSRF. As form will be submitted to `POST /csrf` route, we will create `POST /csrf` to verify CSRF.

```php
use Nishchay\Security\CSRF;

/**
* @Route(path="test", type=[PUT,POST])
* @Response(type=null)
* @return array
*/
public function testPost()
{
    $csrf = new CSRF('test-form');
    if ($csrf->verify()) {
        echo 'SUCCESS';
    }
}
```

###### Step 4: Test it

Now follow below step to check if its working or not

1. Go to `GET /csrf` route.
2. Submit form
3. Output should be `SUCCESS`.

To check for failure of verification call `POST /csrf` without any token or with invalid token. Route will throw `Nishchay\Exception\BadRequestException` exception.