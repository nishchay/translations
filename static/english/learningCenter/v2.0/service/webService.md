#### Web Service

Nishchay provides web service features apart form RESTful route. Route allow us to create SEO friendly URL which can also used to create RESTful API. Using allowed method type we can also create GET, POST, PUT or other service to provide various operation on same route. Using response type we can create JSON/XML response.

Apart from route path and HTTP method, Nishchay provides various features like defining error and success response structure, filtering response, token based service and many other.

###### Create Service

To make route a service, annotate that route with `@Service` annotation. All parameters of this annnotation are optional which allow us to specify empty annotation definition.

**NOTE:** When route act as service it must have response type JSON or XML.

`@Service` supports following parameters.

| Name      | Description                                                                      |
| --------- | -------------------------------------------------------------------------------- |
| fields    | Enable or disable field demand or specify list of allowed fields demand in array |
| token     | Enable or disable token for the service                                          |
| supported | List of supported fields                                                         |
| always    | List of fields which will always be returned                                     |

###### Make whole application serve as Service

To make whole application serve as Service, update setting to TRUE of _enable_ config in _service.php_ file. This will make all routes a service. We don't need to specify `@Service` annotation on each route but we can still define it if want to configure something for service.

###### Filter service response/Fields demand

Client can request which fields a service should respond with by sending list of field names separated by comma in 'fields' GET parameter.

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

If we call above service by `user/1234` it will respond with all fields returned by route method. If we request same service with `fields` GET parameter it will be fitered based on fields received. For request `user/1234?fields=name,age`, it will respond with only two fields _name_ and _age_.

###### Force fields demand

By default demanding fields is optional. Omitting it will respond with everything. But we can make fields demand required by setting `fields` parameter of `@Service` annotation to `null`.

```php
#[Service(fields: null)]
```

Calling service without fields GET parameter where fields demand is required, service results in an error.

###### Disable fields demand

Setting _fields_ to _FALSE_, disables fields demand for service. Requesting fields on this service results in an error.

```php
#[Service(fields: false)]
```

###### Specify default fields demand

If value of _fields_ parameter is array, it will act as default fields demand. If this is set and client passes fields GET parameter then it overrides default demand.

```php
#[Service(fields: ['name', 'age', 'gender', 'accountType'])]
```

###### Point to note for fields demand

If route method does not return requested fields then those fields will be added with default value as specified in service config. Nishchay add those fields with default only if its present in always returning and supported. Same will result in an error if its doesn't exist in route response, always returning and supported.

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

For above request `user/1234?fields=name,gender,email` will respond all requested fields but for email field it will be default value. Because `email` is supported by service but not returned by route method.

If `user/1234?fields=name,email,birthDate` then it will result in an error because its not supported by service and also its not returned from route method.

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

We can make service to always respond with fields. Even if fields demand does not have fields specified in always returning list, it will still respond.

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

Services are considered as secured if it requires to token to access it. Some service requires authentication to access it. To access those service client might need to send username and password in header/GET parameter to access it. Nishchay web service provides very familiar way. We should implement non secure service, where user can pass credential. Once user verified we can generate token and respond to user so that user can use this token to access secure service.

By default all services are secured, we can make specific service non secure by setting `token = true` of `@Service` annotation. We can also make all service non secure by setting `token = false` in config file.

###### Where token should be passed

Client can send token in header or GET parameter. This location is configurable to header and GET parameter only from `token.where` config in `service.php` setting file.

###### What should be token name

By default token is configured to be sent in header with `X-Service-Token` name. As mentioned above we can change token location to GET, in this case we might need to change token name. If we set `token.name` in `service.php` setting to `token`, then token should be passed like this `user/1234?token={token}`.

###### Persisting token

Token should be persisted so that it can checked on further requests. Nishchay checks for token in session with name specified in `token.seasonName` config.

There's way to save token to this session which is discussed in below section. Default name for session is `serviceToken`. Token is saved in Nishchay internal session which is not accessible to developer.

If we store token in database, then it should be fetched from database then set it to this session. You should do this in event which gets called before each secure service.

##### Service class

Using Service class we can know if the field is requested by client or not. We can also get or set service token in session.

###### Service instnace

Instance of service class is assigned to controller class property by applying `@Service` on it.

```php

#[Service]
private $service;
```

Before calling route method, Nishchay will instnace of `Service` class to `$service` property. Here we can have any name for property, Nishchay assigne instnace based on annotation defined on property.

###### Check if field is demanded

Checking if field is requested by client or not.

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
