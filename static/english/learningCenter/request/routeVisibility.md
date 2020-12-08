#### Route visibility
There can be routes in application should be visible based on user agent, ip address and time. Application might have routes which should be visible only user agent(e.g. firefox or chrome), or there can be multiple scenario based on that route should be visible. 

Route visibility is for specifying visibility of routes based on list of scope, time, user agent, IP addresses and response of callback method.

We do not have to define visibility on each route or controller but this can defined in `route.php` setting file. Then on each request nishchay framework checks for its visibility if its applicable.

We can create any number of visibility we want. Below is visibility config:

```php
'visibility' => [
    /**
    * Enable or disable
    */
    'active' => true,
    'config' => [
        '{visibilityName}' => [
            /**
            * Enable or disable this configuration.
            */
            'active' => true,
            'eligible' => [
                /**
                * List of scopes to which this visibility setting will be applied.
                */
                'scope' => [
                    '{scopeName}'
                ]
            ],
            /**
            * Define single or multiple visibility conditions.
            */
            'visible' => [
                [
                    /**
                    * Visible for given time ranges.
                    */
                    'time' => [
                        ['00:00', null]
                    ],
                    /**
                    * Visible for given agents
                    */
                    'agent' => [
                        '{userAgent}'
                    ],
                    /**
                    * Visible for list of IPs
                    */
                    'ip' => [
                        '127.0.0.1'
                    ],
                    /**
                    * Visible if given callback returns TRUE.
                    */
                    'callback' => function() {
                        return true;
                    }
                ]
            ]
        ]
    ]
]
```

Let's go through each settings in visibility configuration.

###### active
To disable or enable visibility config. When disabled nishchay does not check for this visibility.

###### eligible
Here we can specify list of scopes. Visibility configuration will be applied to all routes belongs to specified scopes.

###### visible
This setting manages its visibility. Here we can specify time range, agent list, IPs and callback method.

If all of the visibility matched then route will be visible. If we do not specify visibility setting of one or more, then route will be visible based on specified visibility setting only.

Suppose we have defined visibility of time and agent then route will be visible only if time and agent is matched, ip and callback visibility won't be considered.

###### visible - time
Here we can specify list of times. This time can be `H:i` or `Y-m-d H:i` format. When time range is defined in `H:i` then it will be applicable for all day. Within each range we can specify start and end time.

If we specify only start time then visibility will be after given time and it won't end. Similarly specifying only end time will last for given end time only.

For daily at given time range

```php
['21:00','22:00']
```

After given time
```php
['21:00']
```

After future date

```php
['2021-09-09 21:00']
```

For future date range

```php
['2021-09-09 21:00', '2021-09-11 23:59']
```

###### visible - agent
Here we can specify list of user agents for which routes needs to be visible. This visibility can be applied to range of agent version or version greater or less than range also.

If we do not specify version then it will be applicable for all versions of user agent.

Match exact version
```php
['firefox/52.0']
```
Version range
```php
['firefox/45.0-50.0']
```
This will match firefox having version number from 45.0 to 52.0.

Version less or equal
```php
['firefox/-52.0']
```
This will match browser version from 0 to 52.0

Version equal and above
```php
['firefox/52.0-']
```

###### visible - ip
Here we can specify list of IPs for route visibility.
```php
['127.0.0.1']
```

###### visible - callback
We can have callback which should return `true` if route needs to visible. Response of callback method other than `true` is considered as `false`.

```php
'callback' => function() {
    return true;
}
```