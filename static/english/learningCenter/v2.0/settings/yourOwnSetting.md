#### Your Own Setting

To create our own setting which might be required for application, place php file inside `settings/configuration` directory. Setting file must return array just like as shown below:

```php
return [
    'githubApiKey' => 'GITHUB_API_KEY',
    'googleMapKey' => 'GOOGLE_MAPS_KEY'
];
```

Let's call this file `credentials.php`. To fetch value of `githubAppKey` from this setting file, use `getSetting` method of `Nishchay` facade class. See below code for how to fetch setting.

```php
Nishchay::getSetting('credentials.githubAppKey');
```

If settings are nested then separate it by dot where first part should be file name without its extension. Let's change above setting to understand how nested can read.

```php
return [
    'github' => [
        'apiUrl' => 'https://api.github.com/',
        'apiKey' => 'GITHUB_API_KEY'
    ]
];
```

Now to read `apiKey` we need to change our parameter to method `getSetting`. 

```php
Nishchay::getSetting('credentials.github.apiKey');
```

