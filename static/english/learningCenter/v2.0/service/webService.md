#### Web Service

Nishchay provides web service features apart form RESTful route. Route allow us to create SEO friendly URL which can also used to create RESTful API. Using allowed method type we can create GET, POST, PUT or other service to provide various operation on same route. Using response type we can create JSON/XML response.

Apart from route path and HTTP method, Nishchay provides various features like defining error and success response structure, filtering response, token based service and many other.

###### Create Service

To make route a service, declare `Nishchay\Attributes\Controller\Method\Service` on controller method.

**NOTE:** When route act as service it must have response type JSON or XML.

`Service` attributes has following parameters.

| Name      | Description                                                                      |
| --------- | -------------------------------------------------------------------------------- |
| fields    | Enable or disable field demand or specify list of allowed fields demand in array |
| token     | Enable or disable token for the service                                          |
| supported | List of supported fields                                                         |
| always    | List of fields which will always be returned                                     |

###### Make whole application serve as Service

To make whole application serve as Service, update setting to TRUE of _enable_ config in _service.php_ file. This will make all routes a service. We don't need to declare `Service` attribute on each route but we can still declare it if want to configure something for service.

###### Filter service response/Fields demand

Service responds various fields but in some case does not want all fiends sent by service. If service consumer passes _fields_ parameter then nishchay filters out fields before sending it to consumer. This _fields_ parameter must be _GET_ parameter.

```php

#[Service]
#[Route(path: 'user/{userId}')]
#[Placeholder(['userId' => 'int'])]
#[Response(type: 'json')]
public function userProfile($userId = "@Segment(index='userId')") {
   return [
       'name'=>'Bhavik Patel',
       'age'=>28,
       'gender'=>'Male',
       'accountType'=>'Developer'
   ];
}
```

If we call above service by _user/1234_ it will respond with all fields returned by route method. If we request same service with `fields` GET parameter it will be filtered based on fields received. For request _user/1234?fields=name,age_, it will respond with only two fields _name_ and _age_.

In _fields_ parameter, multiple fields should separated by comma.

###### Force fields demand

By default demanding fields is optional. Omitting it will respond with everything. But we can make fields demand required by setting `fields` parameter of `Service` attribute to _null_.

```php
#[Service(fields: null)]
```

In this case, when consumer calls service without _fields_, it responds with error.

###### Disable fields demand

Setting _fields_ to _FALSE_, disables fields demand for service. Requesting fields on this service results in an error.

```php
#[Service(fields: false)]
```

###### Specify default fields demand

If value of _fields_ parameter is array, it will act as default fields demand. If this is set and consumer passes fields GET parameter then it overrides default demand.

```php
#[Service(fields: ['name', 'age', 'gender', 'accountType'])]
```

###### Point to note for fields demand

If route method does not return requested fields then those fields will be added with default value as specified in service config. Nishchay add those fields with default only if its present in _always_ and _supported_. Same will result in an error if its doesn't exist in route response, _always_ and _supported_ parameter of `Service` attribute.

```php

#[Service(always: ['name', 'gender'], supported: ['name', 'email', 'age', 'gender', 'accountType'])]
#[Route(path: 'user/{userId}')]
#[Placeholder(['userId' => 'int'])]
#[Response(type: 'json')]
public function userProfile($userId = "@Segment(index='userId')") {
   return [
       'name'=>'Bhavik Patel',
       'age'=>28,
       'gender'=>'Male',
       'accountType'=>'Developer'
   ];
}
```

For above request `user/1234?fields=name,gender,email` will respond all requested fields but for email field default value will be applied. Because `email` is supported by service but not returned by route method.

If `user/1234?fields=name,email,birthDate` then it will result in an error because _birthDate_ not supported by service and also its not returned from route method.

###### Define supported fields

Defining supported fields for service will make sure service should respond with only supported fields. If in case method respond with fields which are not specified in supported list, then it will be removed by Nishchay service post processor.

```php

#[Service(supported: ['name', 'email', 'age', 'accountType'])]
#[Route(path: 'user/{userId}')]
#[Placeholder(['userId' => 'int'])]
#[Response(type: 'json')]
public function userProfile($userId = "@Segment(index='userId')") {
   return [
       'name'=>'Bhavik Patel',
       'age'=>28,
       'gender'=>'Male',
       'accountType'=>'Developer'
   ];
}
```

For above `gender` field will be removed as its not supported by service.

###### Always returning fields

We can make service to always respond with fields. Even if fields demand does not request fields specified in _always_ list, it will still be responded.

```php

#[Service(always: ['name', 'gender'])]
#[Route(path: 'user/{userId}')]
#[Placeholder(['userId' => 'int'])]
#[Response(type: 'json')]
public function userProfile($userId = "@Segment(index='userId')") {
   return [
       'name'=>'Bhavik Patel',
       'age'=>28,
       'gender'=>'Male',
       'accountType'=>'Developer'
   ];
}
```

For request `user/1234?fields=age` will respond with name,age and gender fields. `name`, `gender` fields are not demanded but are still responds as its set to always respond.

##### Secure Service

Services are considered as secured if it requires to token to access it. Some service requires authentication to access it. To access those service consumer might need to send username and password in header/GET parameter to access it. Nishchay web service provides very familiar feature. We should implement non secure service, where user can pass credential. Once user verified we can generate token and respond to user so that user can use this token to access secure service.

By default all services are secured, we can make specific service non secure by setting `token = true` of `Service` attribute. We can also make all service secure or non secure by setting _token_ in _service.php_ config file.

###### Where token should be passed

Consumer can send token in header or GET parameter. This location is configurable to header and GET parameter only from _token.where_ config in _service.php_ setting file.

###### What should be token name

By default token is configured to be sent in header with _X-Service-Token_ name. As mentioned above we can change token location to GET, in this case we might need to change token name. If we set _token.name_ in _service.php_ setting to _token_, then token should be passed like this _user/1234?token={token}_.

##### Service Token

###### Generate

There's two ways to create service token.

1. OAuth
2. Custom

For OAuth use `Nishchay::getOAuth2()` to generate its token. Configuration related to this type of token is located in _service.php_ setting file. When you want to use this token set _verifyCallback = 'oauth'_. This is default setting.

Below example demonstrate how we can generate oauth token.

```php
$userId = 1;
$scopes = ['account', 'notes'];
$token = Nishchay::getOAuth2()->generateUserCredentialToken($userId, $scopes);
```

Here first parameter should be logged in user's userId and second parameter should be list scope names (As defined on route).

**Custom**

Another is one custom token, which you can generate on your own. How this token will be verified, please read next section.

###### Verification

If you are using Nishchay's OAuth token then you don't have to do anything as nishchay verifies this token for secure service. It also verifies scope.

When you generate OAuth token you also list of scope names. This will make token to be valid for given scopes. Upon requesting secure service, if it has scope defined on called route then nischay will check scope mentioned on route does exists in OAuth token.

For above example(See section Generate), token will be valid of _account_ and _notes_ scope.

###### Custom verification

If you are not using Nishchay OAuth token then you can specify callback method in _token.verifyCallback_ of _service.php_ setting file. When this setting has callback function set, then this function called with token received from request. Within this function you can verify your custom token.

This method must return _true_ if token verification has been passed.

##### Service class

Using Service class we can know if the field is requested by consumer or not.

###### Service instance

Instance of service class is assigned to controller class property by declaring `Nishchay\Attributes\Controller\Property\Service` attribute on it.

```php

#[Service]
private $service;
```

Before calling route method, Nishchay will create instance of `Service` class and assigns to `$service` property. Here we can have any name for property, Nishchay assigns instance based on attribute declared on property.

###### Check if field is demanded

Checking if field is requested by consumer or not.

```php
$this->service->getNeed()->isFirstName()
```

Any field can checked by `is{fieldName}`, here first character of fieldName should be upper case. So for above, `FirstName` will be checked as `firstName`.

###### Save token to session

Saving service token.

```php
$this->service->setToken('TOKEN')
```

###### Get service token

Getting service token.

```php
$this->service->getToken()
```
