#### Maintenance modes

Nishchay provides various ways to put application into maintenance. Maintenance configuration file is located at `settings/configuration/maintenance.php`. Nishchay processor check for maintenance only when it is enabled.

There are three ways application can be put into maintenance which are time, user agent and callback. 

There must be route which will be called for maintenance mode. For each type of maintenance mode there must be default route which will be called if any mode config does not specify its own route.

##### Enable

To enable maintenance check, set `active = true`. This will enable maintenance mode but still we need to enable at least one of mode. By default each modes are disabled.

##### Timed

We can put application into maintenance by giving start and end time which can be time only or full date with time. Setting time only, will make it enabled for every day. Here time only format is `H:i` and full date format is `Y-m-d H:i`. Timed mode configuration is contained in `time` setting. 

Timed config is shown below:

```php
[
    'active' => false,
    'list' => [
        [[START, END], ROUTE, MAINTENANCE_MESSAGE]
    ],
    'default' => 'maintenanceRoute'
]
```
Timed maintenance mode configuration contains three settings which are listed below:

| Name | Description |
| ----- | ----- |
| active | To enable this mode |
| list | List of maintenance time slot |
| default | Default route in case there's no route defined in each time slot |


###### Time slot
 Each slot should be array where its elements should be.
 
 1. Time slot in an array where its first element is start time and second element is end time.
 2. Route to call for this maintenance time slot.
 3. Maintenance message for this slot


Below example shows how we can put an application into maintenance for each day from 12:00 to 13:00.

```php
[['12:00', '13:00']]
```

As per because we have specified route to be called for this slot, Nishchay will look for default route for timed mode. If there's no defualt route defined for timed mode then route specified in `route` will be used.

##### Client User Agent

Nishchay also allow us to put application by list of client user agents. Configuration for this mode is in `agent` setting which are listed below:

| Name | Description |
| ----- | ----- |
| active | To enable this mode |
| invert | When this is `true`, maintenance will not be activated for browser mentioned in `list` |
| list | List of browesers with its versions |
| default | Default route in case there's no route defined in each list |

User agent config is shown below:
```php
[
    'active' => true,
    'invert' => false,
    'list' => [
        [BROWSER_NAME/{VERSION}, ROUTE, MAINTENANCE_MESSAGE]
    ],
    'default' => 'maintenanceRoute'
]
```
By default if we do not specify version, it matches for all version.

###### Match exact version
```php
["firefox/52.0", "routeToCall", "Maintenance message"]
```
###### Version range
```php
["firefox/45.0-50.0", "routeToCall", "Maintenance message"]
```
This will match firfox having version number from 45.0 to 52.0.

###### Version less or equal
```php
["firefox/-52.0", "routeToCall", "Maintenance message"]
```
This will match browser version from 0 to 52.0

###### Version equal and above
```php
["firefox/52.0-", "routeToCall", "Maintenance message"]
```
This will match browser version from 52.0

##### Callback

We can set callback for maintenance check, this must return boolean value. Enable maintenance mode by returning true. We can have multiple callback. Callback configuraiton is listed below

| Name | Description |
| ----- | ----- |
| active | To enable this mode |
| list | List of callback in an array |
| default | Default route in case there's no route defined in each list |

Config for this mode is shown below:
```php
[
    'active' => FALSE,
    'list' => [
        [CALLBACK, ROUTE, MAINTENANCE_MESSAGE]
    ],
    'default' => 'maintenanceRoute'
]
```

##### Allowed

While in maintenance mode we can also allow specific route to be called by specifying its context, route name or scope. This can be enabled or disabled based on `ignoreAllowed` flag. By default this flag set to false, making allowed to checked. When `ignoreAllowed` is set to `true`, allowed list won't be considered and there won't be any allowed routes.

###### Context

We can allow one or more context even in maintenance mode. All routes belongs to listed context are accessible.

```php
[
    'active' => TRUE,
    'list' => [
        'Application/Access/Direct'
    ]
]
```
Above will make all routes belongs to `Application/Access/Direct` context to be accessible even in maintenance mode. 

###### Route

One or more routes can be added to exception list so that it can still be executed in maintenance mode. There are three ways we can ignore routes for maintenance mode.

1.  Fixed match

Routes defined as fixed match will be matched exact same.

2.  Prefix match

Any routes start with specified prefix will be matched.

3.  Regex

It will allow us to create our regex expression to be matched.

Just like context ignore mode, this can also be enabled or disabled. Below code demonstrates regex match.
```php
[
    'active' => TRUE,
    'list' => [
        [
            'type' => 'regex',
            'match' => '^special/(\d+)$'
        ]
    ]
]
```
###### Scope

We can add one more scope to be allowed in maintenance mode.

```php
    [
        'active' => TRUE,
        'list' => [
            'help'
        ]
    ]
```

As per above code any route which have scope `help` will be accessible in maintennace mode.