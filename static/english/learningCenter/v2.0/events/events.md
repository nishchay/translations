#### Events

Events are executed before or after execution of route. There are four times when events are executed which are listed below.

1.  Before creating instance of controller
2.  Before calling route method
3.  After calling route method
4.  Before destroying controller instance

Events which executes before or after controller are executed for all routes of controller. These type of event can be defined on controller.

Events can be defined on controller, route or within event class. Event class is class which have all events. Within this event class, we can define events for scopes, context or global events.

##### Event class

We can create separate class to define events. Events class contains three type of events called global, scope and context. Global events are executed for each route while scope & context events are executed for their scope and context only. We can create multiple event of same type.

As there can be multiple events of different type, these events are executed in following order by default.

1.  Global
2.  Context
3.  Scope

This order can be changed for controller or method by event attribute defined on them.

Each methods in event class must have `Nishchay\Attributes\Event\EventConfig` attribute, to make it event method.

This attribute supports following parameters.

| Parameter | Description                                              |
| --------- | -------------------------------------------------------- |
| type      | Can be any of three types `global`, `scope` or `context` |
| when      | When to execute event. This can be `before` or `after`   |
| name      | Name is required for event type scope and context        |
| once      | Makes event to be executed only once during the request. |

Below few example demonstrates how each type of events can be created.

###### Global Event

This will be applied for each routes and it can be executed before each route method is called.

```php
/**
* #[EventConfig(type: 'global', when: 'before')]
*/
public function globalEvent() {
    // Global setup/checkup
}
```

###### Context Event

```php
/**
* #[EventConfig(type: 'context', name: 'Application\Controllers\Direct', when: 'before')]
*/
public function contextEvent() {
    // Context specific setup/checkup
}
```

This will create event for context `Application\Controllers\Direct` and it will be execute before execution of routes which are belongs to mentioned context.

###### Scope Event

Below will create event for scope named _secure_.

**Example**

Suppose we want to check if user is logged in or not, we will create route with attribute `Nishchay\Attributes\Controller\Method\NamedScope` of _secure_.

Below we will create event for _secure_ scope and within it we will check if user is logged in or not.

```php
/**
* #[EventConfig(type: 'scope', name: 'secure', when: 'before')]
*/
public function scopeEvent() {
    // Scope specific setup/checkup
}
```

##### Declaring event on controller or route method

Events are created using `Nishchay\Attributes\Controller\Event\BeforeEvent` and `Nishchay\Attributes\Controller\Event\AfterEvent` attribute. Both attributes supports same parameters. Event defined using this attribute have priority then event defined in event class, this means events defined on controller or route method calls first.

Parameters supported by above two attributes are listed below:

| Parameter | Description                                                                                                                                    |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| callback  | Should be method name only if callback is of same class or it can be `{class}:{method}` if it belongs to another class.                        |
| order     | Optional parameter. Order in which events of event class should be executed. Order list should be any of these `global`, `context` and `scope` |
| once      | Optional parameter. To make event to be executed only once during the lifetime of request. Value of this parameter should be boolean.          |

Below example will create event for all routes within controller and it will also change the order of execution of events of event class. This event will be executed only once as `once` parameter set to true.

```php
/**
* #[Controller]
* #[BeforeEvent(callback: 'beforeEventMethod', order: ['scope','context','global'], once: true)]
*/
class AccountController {
```

##### What event should return

Response of event affect the execution of next event if exist and route. If the event returns true it will allow next event or route to be executed.

If it returns `false` then request results in `BadRequestException`.

If the event does not return boolean value then this response will be counted as the response of route. In this case route will not be executed and event response will be sent as an request response.

**NOTE** To denote success of event, it must return `true`.
