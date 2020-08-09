#### Response Handling

Response reuturned by route method considered as request response. By default response is view and it can be changed to JSON, XML or NULL.

##### View

When response type is view, route should return single view name as stirng or list of view name as array.

``` php
/**
 * @Route(path='namaste')
 */
public function namaste() {
    return 'namaste';
}
```

Once response is returned by route, it handed over to response handler. It locates view in all views directory found in application and renders it. If route returns more than one view then it gets rendered one by one in same order.

###### Passing data to view

To pass data to view use [RequestStore](/learningCenter/request/requestStore)

##### How view is located

###### Context

Response handler first finds view in current context. Suppose route is located in controller class _Application/Controllers/Access/Direct/WelcomeController_ and you have defined controller to be inside _Application_ directory then _Application_ becomes context of route.

Response handler will first find view in _Application/views_ directory as _Application_ is context of the route being processed.

If it does not found there then it tries to locate by going up.

###### Going up method

Suppose current context is _Application/Access/Direct_ then view will be searched as per first method called context. If it not found thencontext gets removed one at time then it gets located and search is proceed as shown below.

1.  _Application/Access/Direct/views_
2. Now `Direct` directory will be removed form path and response handler will find view in `Application/Access/views` .
3. Now `Access` directory will be removed from path and response handler will find vie win `Application/views` 

It ends when there no more directory exists in path or path becomes empty.

###### Collection Lookup

If both of the above step fails, then view is search in all registered view directory within application. View directory gets registered when Nishchay validates structure.

These directory are stored as defined in structure definition.

##### Set response type of route

We can specify response type for whole application in `response` setting. But if some routes need to respond with another type than defined in `response` config, we can use `@Response` annotation.

`@Response` annotation accepts only one parameter called `type` which should have one of following values.

*   view
*   json
*   xml
*   null: to respond with anything we want.

``` php
/**
 * @Route(path='namaste')
 * @Response(type=view)
 */
public function namaste() {
    return 'namaste';
}
```

This will make `namaste` route to respond with view only.

##### Set response type globally

Default response type of all route can be changed from setting file `settings/configuration/response.php` .

##### Response Type: JSON

When route is to respond with json, route resonse must be an array. Response handler converts array into json and sets content type to json.

``` php
/**
 * @Route(path='user')
 * @Response(type=JSON)
 */
public function user() {
    return [
        'name'=> 'Bhavik Patel',
        'age'=> 28,
        'gender'=> 'Male',
        'accountType'=> 'user'
    ];
}
```

As per above code, `user` route will response with: 

``` json
{
    "name":"Bhavik Patel",
    "age":28,
    "gender":"Male",
    "accountType":"user"
}
```

##### Response Type: XML

Just like JSON response for XML response, route must respond with array. If we change response type JSON to XML in above code it will respond with following:

``` xml
<root>
    <name>Bhavik Patel</name>
    <age>28</age>
    <gender>Male</gender>
    <accountType>user</accountType>
</root>
```

###### Set element attribute which has child element

If array has `@attributes` key, its considered as attribute list. Value of this key must be array.

``` json
[
	    'user'=>'bhavik',
	    'name'=>'Bhavik Patel',
	    'age'=>28,
	    'addresses'=>[
	        [
	            '@name'=>'address',
	            '@attributes'=> [
	                'id' => 12345
	            ],
	            'landmark'=>'Landmark',
	            'line1'=>'Address Line 1',
	            'line2'=>'Address Line 2'
	        ],
	        [
	            '@name'=>'address',
	            '@attributes'=> [
	                'id' => 12346
	            ],
	            'landmark'=>'Landmark',
	            'line1'=>'Address Line 1',
	            'line2'=>'Address Line 2'
	        ]
	    ]
]
```

Here `addresses` have two child which named `address` . Because putting address as key will prevent adding multiple child node with name. So `@name` will allow us to set name for node. This will output:

``` xml
<root>
        <user>bhavik</user>
        <name>Bhavik Patel</name>
        <age>28</age>
        <addresses>
            <address id="12345">
                <landmark>Landmark</landmark>
                <line1>Address Line 1</line1>
                <line2>Address Line 2</line2>
            </address>
            <address id="12346">
                <landmark>Landmark</landmark>
                <line1>Address Line 1</line1>
                <line2>Address Line 2</line2>
            </address>
        </addresses>
</root>
```

###### Set element attribute which has value

To set value of an element which does not child element, we can set `@value` to sepicfy value for that element. See below code:

``` json
[
    [
        '@name' => 'name',
        '@attributes' => [
            'id' => 12345
        ],
        '@value' => 'User full name'
    ]
]
```

which results in following response:

``` xml
<root>
    <name id="12345">User full name</name>
</root>
```

##### Response Type: Nothing

Setting response type to _NULL_ will allow us to set any response type we want. If response type is null then based on response of route, response type is decided. This is how it works.

1. If route returns string, it simply returned as response.
2. To return view follow this format: `view:{viewName}`
3. When route returns an array, then response handler generates json response.
4. Returning nothting doesn't do anything.

##### Set response header

Static method `Nishchay\Http\Response\Response::setHeader` allow us to set response header. It supports two parameter first being header name and second being header value.

``` php
Response::setHeader('Cache-Control', 'no-cache, must-revalidate')
```

Response class has dedicated method to set Content type and response status. 

###### Respons status code
To set response status code just pass response code as first parameter on `setStatus` method of `Response` class. We can pass message as second paramer to set reponse status message, omitting it will use default message.

``` php
Response::setStatus(404)
```

###### Set Content Type

Using `setContentType` we can set content type of response. This is required in the case route response type is `null`, as set in `@Response` annotation.

``` php
Response::setContentType('application/json')
```

We can also pass alias of response type instead of passing full type name. Such alias are listed below.

| Alias | Content Type | Use           |
|-------|--------------|---------------|
| html  | text/html    | Standard HTML |
| plain  | text/plain    | Simple raw text |
| text  | text/text    | Simple raw text |
| json  | application/json    | JSON data content |
| js  | applicaiton/javascript    | Javascript content |
| javascript  | applicaiton/javascript    | Javascript content |
| xml  | text/xml    | XML data content |
| png  | image/png    | PNG image |
| jpg  | image/jpg    | JPG image |
| jpeg  | image/jpeg    | JPEG image |
| gif  | image/gif    | GIF image |