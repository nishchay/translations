#### Localization

Nishchay provides localization feature to translate application into multiple languages. We only have to create translation file which contains translation of each statement in an application. These statement either known by its key or statement itself. Below example demonstrate translation file.
```json
{
  "home": "Welcome to my application",
  "user": {
    "label": {
      "firstName": "First name",
      "lastName": "Last name"
    },
    "error": {
      "firstName": {
        "required": "Please enter first name",
        "invalid": "Please enter valid first name"
      }
    }
  }
}
```
We can have as my translation as an application needed to be supported. Translation is know by its file name. If file name is `english.json`, then its know by `english`.

##### Settings

For the translation related setting please read [here](/learningCenter/settings/lang).

##### Printing translation

Statement is printed using `line` method of `Lang` class. `Lang` is singleton class whose instance created when its first usage is made. We can get instance by using `Nishchay::getLang()` method.

```php
# To print translation of 'home' from file mentioned above.
Nishchay::getLang()->line('home');

# Nested statements are chained by dot(.).
Nishchay::getLang()->line('user.label.firstName');

Nishchay::getLang()->line('user.error.firstName.required'); 
```
##### Set Default and main locale

To set default or main locale for an application, we only have to pass file name without extension as method argument. Set main locale.
```php
Nishchay::getLang()->setMain('hindi');
```
To set default locale.
```php
Nishchay::getLang()->setDefault('english');
```
Suppose our application needed to support hindi, italy and english language, then we have to create translation file which can be named as `hindi`, `italy` and `english`. If user is accessing application from india, we will set our main locale to hindi and default locale to `english`. Its advice to set default locale as `english`.

##### Namespace

We can group translations into multiple file and these files can also stored as nested hierarchy. Main directory will be known as locale name its subdirectory and files are known as namespace.

Suppose we want our application's translation to be stored as per each pages. We can file for each pages. See below hierarchy:

```yaml
english
    - pages
        - home.json
        - login.json
        - register.json
        - account.json
    - english.json
```

For above hierarchy, to fetch translation stored in `home.json` file `pages/home` should be passed as namespace and translation to be passed as usual. Suppose for example there's title for home page stored in `english/pages/home.json` we can do as follows:

```php
Nishchay::getLang()->line('title', [], 'pages/home');
```

If locale and file stored on root directory is same then it will serve as root namespace. In this case we don't need to pass any namespace while fetching translation.

##### Fallback/Default locale

If translation of given key does not found in main locale then translation is fetched from default locale. If application does not support multiple language, then main and default locale should be same.

##### Placeholder

Some translation may have dynamic value which can be replaced by using placeholder. Placeholder can be any name or it can numeric number too. Suppose we have following translation.
```php
[
    'hiWithName' => 'Hi, {name}'
]
```
Here _name_ is placeholder whose value depends on parameter we pass during printing sentence. Below is code to show how we can pass placeholder value.
```php
Nishchay::getLang()->line('hiWithName',['name' => 'Full Name'])
```
###### What if placeholder not found or not passed?

Placeholder won't be replaced if we do not pass placeholder or passed placeholder does not exist in sentence. Sentence will be returned placeholder unchanged.

###### Escape placeholder

We can also escape placeholder by prefix placeholder with `@`.
```php
[
    'demo' => 'Here @{name} will be replace with {name}.'
]

// Here first placeholder won't be replaced but second will.
echo Nishchay::getLang()->line('demo', ['name' => 'Full Name']); // Prints: Here {name} will be replaced with Full Name.
```