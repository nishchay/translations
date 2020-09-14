#### Read Setting

###### Application

Application setting contained in application.php file can be read using two methods `getConfig` or `getSetting` of Nishchay facade class.
```php
Nishchay::getConfig('application.name')
```
Method `getConfig` is for reading only application setting while `getSetting` we can read any setting. Method `getSetting` method requires config path to be prefixed by its file name. 

To read `application.name` setting using `getSetting` method:

```php
Nishchay::getSetting('application.application.name')
```
###### Reading other setting

Settings other than `application.php` are read by using  `getSetting` method. We can read any setting contained in `settings/configuration` directory.

Let's take an example that we have setting file called: `credentials.php`. This setting contains following configuration:

```php
return [
    'github' => [
        'apiUrl' => 'https://api.github.com/',
        'apiKey' => 'GITHUB_API_KEY'
    ]
];
```

Reading `apiKey` from above setting:

```php
Nishchay::getSetting('credentials.github.apiKey');
```

We can also read setting which have array of values.

```php
$github = Nishchay::getSetting('credentials.github');
```

Above  will return both `apiUrl` and `apiKey` as object. As we have stored github config in `$github` variable, its setting can be read as follows:

```php
echo $github->apiUrl;
```

###### Other setting methods

There are various methods in Nishchay class which allows us to directly check for application stage name or if application is live or not and etc. Such methods are listed below

| Method name | Description |
| ----- | ----- |
| getApplicationAuthor | Application author name |
| getApplicationName | Application name |
| getApplicationStage | Current application stage |
| getApplicationVersion | Current application version |
| isApplicationStageLocal | Returns `true` if current applicaiton stage is `local` |
| isApplicationStageTest | Returns `true` if current applicaiton stage is `test` |
| isApplicationStageLive | Returns `true` if current applicaiton stage is `live` |
| getSupportedExtension | Returns supported file extensions as array |