#### Application Logging

Nishchay provides various ways to logging application errors and info. Log can be stored in either file or database based on configuration. Log related settings are contained in `settings/configuration/logger.php` setting file. You can learn more about logger setting [here](/learningCenter/settings/logger).

##### Logging types

Application logger have various methods to log message, such as `log` is for logging errors. Other logging methods are listed below.

*   error
*   warning
*   notice
*   info

All of above methods accepts only one argument which should statement of log. Logger automatically prepends time and log type with log statement while writing log message into file or db.

###### Example
```php
// Logging error
Nishchay::getLogger()->error('This should be error statement');

// Logging warning
Nishchay::getLogger()->warning('This should be warning statement');

// Logging notice
Nishchay::getLogger()->notice('This should be notice statement');

// Logging info
Nishchay::getLogger()->info('This should be info statement');
```
##### Logging other type

If we want to log statement of another type then `write` should be called with first parameter as log type and second as log statement.
```php
Nishchay::getLogger()->write('typeName','Type statement.');
```
##### Enable/Disable logger

By default application logging is disabled but we can change it by setting _logger.enable_ to true. Logger configuration is located at _settings/configuration/logger.php_.

##### Changing logger file location

Logger default configuration is file with logger path `{ROOT}\logs` and this can be changed to anything path we want by updating `logger.path` setting.

##### Change logging to file or db

Please read [Logger setting](/learningCenter/settings/logger) for changing log location to DB.

##### Log duration/rotation

Application logging for the file generates file based on `logger.duration` setting. If the duration is `date` then everyday new file is generated with that date. And all logs are stored in that file only.

We can change this duration to week, biweek or month. For example if the duration is `month`, log file will be generated for once per month and logs are logged to that file of that month only.

For each duration file is always is generated with starting date with `Y-m-d` format. If the duration is biweek then one file is generated with `Y-m-01` and after day 15, it will create another file for that period.