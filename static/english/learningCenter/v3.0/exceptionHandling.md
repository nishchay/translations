#### Exception Handling

Exceptions are handled at scope (Scopes are defined using `Nishchay\Attributes\Controller\Method\NamedScope` attribute on route method), context (This is based on structure definition, to find any route's context call `Processor::getStageDetail('context')`) or global level in Nishchay. We only need to declare `Nishchay\Attributes\Handler\Handler` on class to make it exception handler class. Handler defined for scope will be executed for all routes which belongs to mentioned scope. Same applies to context exception handler. Global handlers are applies to all routes.

##### How it works

Whenever exception occurs, let's say `RequestNotFoundException`. This occurs when request url does not matches with any route in an application. Nishchay first tries to find handler method with name as exception class in located exception handler class. In this case, Nishchay finds `requestNotFoundException` method, if there's no method then `handlerAll` is called.

If exception handler class does not have `handlerAll` method and method with name as exception class, then default handler(Nishchay's exception handler) is called.

Exception handlers are located in following order, it uses which matches first.

1. Exception handler defined for route
2. Handler belongs to current request scope.
3. Handler belongs to current request context.
4. Global handler
5. If there's no exception handler defined in an application or there's an error in above handler then default exception handler is called.

**NOTE:** Method name must start with lowercase and remaining character unchanged.

##### Handler for route

We can define exception handler for specific route or for all routes within controller. To define exception handler for specific route, declare `Nishchay\Attributes\Controller\ExceptionHandler` attribute on route method. For all routes within controller, define it on controller class.

This attribute have following parameters:

| Parameter | Description                                                                       |
| --------- | --------------------------------------------------------------------------------- |
| callback  | Name of method which needs to be called exception. This must exists in self class |
| order     | To change the order of handler lookup                                             |

**NOTE:** Exception handler defined on route completely overrides handler defined on controller.

Below code demonstrates how we define exception handler on controller. Same is applicable for controller method(route).

```php
/**
* #[Controller]
* #[ExceptionHandler(callback: 'exceptionHandler')]
*/
class NamasteController {

    /**
    * Exception handler for this controller
    */
    public function exceptionHandler(Detail $detail) {
        # Handler exception
    }

}
```

There won't be any further exception if defined callback method does not exists in controller class. In this case default handler will be called.

##### Exception instance

Exception handler method receives instance of `Nishchay\Handler\Detail` class which contains information about exception thrown. Detail class contains following information, all of them accessed by their getter method.

| Method     | Description                                                         |
| ---------- | ------------------------------------------------------------------- |
| getMessage | Exception message                                                   |
| getFile    | Path to file exception occurred                                     |
| getLine    | Line number where exception occurred                                |
| getCode    | Error code for the exception. Returns `0` if there's no error code. |
| getType    | Type of an exception                                                |
| getTrace   | Back trace                                                          |

##### Handler class

Declaring `Nishchay\Attributes\Handler\Handler` attribute on class makes it exception handler class. This attribute supports two parameter using which we can assign this class for `global`, `context` and `scope`.

Below code demonstrates how we can define exception handler for scope `secure`.

```php
/**
* #[Handler(type: 'scope',name: 'secure')]
*/
class SecureExceptionHandler {
//....
```

Supported parameters of this attribute is listed below:

| Parameter | Description                                   |
| --------- | --------------------------------------------- |
| type      | Can be global, context, scope.                |
| name      | Applicable for handler for context and scope. |

Parameter name `name` is required when `type` is `context` or `scope`. This should be name of context, for handler type for context. For the scope handler this should be name of scope.

To define method which can handle exception, it should be either same as exception class name with its first letter in small case. Below method handles exception thrown for `RequestNotFoundException`.

```php
public function requestNotFoundException(Detail $detail) {
    # Handle exception
}
```

We can define as many as handler for exception class.

To define handler for all types of exception define method with name `handlerAll`. This method will be called when method with name as exception class does not exists in handler class.

```php
public function handlerAll(Detail $detail) {
    # Handle exception
}
```

##### Handler response

Handler response must return response type same as response type of route. If it does not returns response type same as route, then further response exception is thrown based on response type of route & handler. After this default exception handler gets called to handle exception thrown because of invalid response type.

To find response type of current request, we can do that by fetching current route method from controller collection. See below code:

```php
$route = Processor::getStageDetail('object');
$response = Nishchay::getControllerCollection()
        ->getClass($route->getClass())
        ->getMethod($route->getMethod())
        ->getResponse()
        ->getType();
```

If exception has occurred before route method is called then calling `Processor::getStageDetail('object')` throws exception. There always be `Nishchay\Http\Response\Response` for all routes. Even if we don't define `Nishchay\Http\Response\Response` attribute on route, Nishchay assigns it with default response for the route.

##### Conflict

When we try to define handler of same type then exception is thrown. Nishchay does not allow duplicate exception handler of same type.

We can have only one global exception handler per application. There should be only one handler per context and scope. Suppose if we have already defined handler for scope named _Secure_, defining handler for this scope then it results in an exception.
