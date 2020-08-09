#### Environment variable

There always configuration which changes according to application settings. Such settings are database credentials, cache configuration and etc. These configurations are always different for local, test and live application stage.

Because each settings file does not allow configuration to be as per application stage. We can store such configuration as per application stage using environment variable.

##### Why environment variable

As these configurations are stored as per application stage then why is it called environment variable instead application stage data?

This is because `Nishchay` first finds data in environment variable then it looks into stages data.

##### Location

These data are stored in into their own file in `settings/env` directory. File named same as application stage name. For application stage `local` file name is `local.ini`.

Below is example of how these data can be stored for `local.ini`, same as applicable for `test.ini` and `live.ini`.

```ini
; Database name
DB_NAME = 'MainDB';

; Host name where database server is located.
DB_HOST = '127.0.0.1';

; Port number on which database server is running.
DB_PORT = 3306;

; Authentication username
DB_USER = 'root';

; Authentication password
DB_PASS = 'root';
```

##### How to fetch

We can these data using `getEnvironment` or `Nishchay::getEnv`. Function `env` shorthand of `getEnvironment`. This method accepts two arguments first being variable name and second one is default(Optional parameter). If there's no data for passed variable, the default is returned. BY default this is `false`.

Fetching data:

```php
$databaseName = getEnvironment('DB_NAME');
```