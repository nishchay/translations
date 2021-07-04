#### Events

Events are executed before or after execution of route. There are four times when events are executed which are listed below.

1.  Before creating instance of controller
2.  Before calling route method
3.  After calling route method
4.  Before destroying controller instance

Events which executes before or after controller are executed for all routes of controller. These type of event can be defined on controller.

Events can be defined on controller, route or within event class. Event class is class which have all events.

##### Event class

We can create separate class to define events. Events class contains three type of events called global, scope and context. Global events are executed for each route while scope & context events are executed for their scope and context only. We can create multiple event of same type.

As there can be multiple events of different type, these events are executed in following order by default.

1.  Global
2.  Context
3.  Scope

This order can be changed for controller or method by event annotation defined on them.

Each methods in event class must have `@Intended` and `@Fire` to make it event method. `@Intended` annotation defines type of event which supports following parameters.

| Parameter | Description |
| --- | --- |
| type | Can be any of three types `global`, `scope` or `context` |
| name | Name is required for event type fo scope and context |

`@Fire` annotation defines when to execute event. This can be before or after. Parameters supported by `@Fire` annotation is listed below:

| Parameter | Description |
| --- | --- |
| when | When to execute event. This can be `before` or `after` |
| once | Makes event to be executed only once during the request. |

Below few example demonstrates how each type of events can be created.

###### Global Event

These events are executed for each routes.
```php
/**
* @Intended(type=global)
* @Fire(when=before)
*/
public function globalEvent() {
    // Global setup/checkup
}
```
###### Context Event

```php
/**
* @Intended(type=context, name="Application\Controllers\Direct")
* @Fire(when=before)
*/
public function contextEvent() {
    // Context specific setup/checkup
}
```

This will create event for context `Application\Controllers\Direct` and it will be execute before execution of routes which are belongs to mentioned context.

###### Scope Event

Below will create event for scope named `secure`. Suppose we want to check if user is logged in or not, we will create route with `@NamedScope` of `secure`. Then we will create event for that scope and within it we will check if user is logged in or not.
```php
/**
* @Intended(type=scope,name=secure)
* @Fire(when=before)
*/
public function scopeEvent() {
    // Scope specific setup/checkup
}
```
##### Defining on controller or route method

Events are created using `@BeforeEvent` and `@AfterEvent`. Both annotation supports same parameters. These events are executed first and then events of event class are executed. Parameters supported by above two annotation are listed below:

| Parameter | Description |
| --- | --- |
| callback | Should be method name only if callback is of same class or it can be `{class}:{method}` if it belongs to another class. |
| order | Optional parameter. Order in which events of event class should be executed. Order list should be any of these `global`, `context` and `scope` |
| once | Optional parameter. To make event to be executed only once during the lifetime of request. Value of this parameter should be boolean. |

Below example will create event for all routes within controller and it will also change the order of execution of events of event class. This event will be executed only once as `once` parameter set to true.
```php
/**
* @Controller
* @BeforeEvent(callback=beforeEventMethod, order=[scope,context,global], once=true)
*/
class AccountController { 
```
##### What event should return

Response of event affect the execution of next event if exist and route. If the event returns true it will allow next event or route to be executed. If it returns `false`, request results in `BadRequestException`. If the event does not return boolean value then this response will be counted as the response of route. In this case route will not be executed and event response will be sent as an request response.

**NOTE** To denote success of event, it must return `true`.