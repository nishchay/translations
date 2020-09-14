#### Validations

Nishchay provides two ways to validate request data either by using `Validator` class and or by using `Form` builder. For the REST service use `Validator` class, if we want to render and also want validation on it use `Form` builder. Both of this same validations.

For Form Builder learn [here](/learningCenter/security/formBuilder).

Creating an instance of `Validator` class.

```php
use Nishchay\Http\Request\Request;

$validator = new \Nishchay\Validation\Validator(Request::POST);
```

Above will create validator class for HTTP POST request. `$validator` will be used to validate POST request only. Here parameter to `Validator` class is optional, by default its `POST` method.

##### Add validation rule

To start with validation, we will add `required` validation rule. `required` validation is for making parameters required. Let's  take an example for login validation. Login usually requires `username` and `password`. Both of these fields are required. When we receive request we always want these fields. See below code how we can make both these required.

```php
$validator->setValidation('username', 'required');
$validator->setValidation('password', 'required');
```

We can also use method chaining:

```php
$validator->setValidation('username', 'required')
          ->setValidation('password', 'required');
```

We can also pass rule for multiple fields in one call

```php
$validator->setValidation([
    'username' => 'required',
    'password' => 'required'
]);
```

When some fields need same validation rule, we can pass fields separated by comma.

```php
$validator->setValidation('username,password', 'required');
```

##### Validate request

Once we add all rules, those rule can be validated for request parameter by calling `validate` method. This method returns `true` or `false` based on validation passed or failed. If there's no validation done then it returns `null`.

Suppose we have created validation for HTTP POST request and if we receive GET request then Validator class does not run validation and it will return `null` indicating that validation did not ran.

```php
if ($validator->validate() === false) {
    # Return error as response
}
```

##### Fetch validation error

Once we validate request using `validate` method and if validation fails, we can get all validation errors by using `getErrors` method.

This will return all error in an array where `key` will be field name and its value will be error message.

```php
$validator->getErrors();
```

If validation has been passed and we still call this method, this will return empty array.

###### Fetch error for the field

We can also fetch error for specific field. If we pass field name in `getErrors` method, it will return error message for the field. If there's no error for the field then this method returns `false`.

```php
$validator->getErrors('username');
```

##### Validation message

By default validator returns default messages for all validation errors. We can override those message for our validation. Updated validation reflects only for that instance only.

```php
$validator->setMessage('username', 'required', 'We need your username');
```

Now when we run validation and `username` parameter not present in request then our new message will be returned for `required` validation rule.

We can also set multiple messages in one call.

```php
$validator->setMessage([
    'username' => [
        'required' => 'We need your username',
    ],
    'password' => [
        'required' => 'We need your password'
    ]
]);
```

When we pass multiple message, `key` should be field name and its value should be list of validation messages for rule.

###### Placeholder message

We can put placeholder in message and it will be replaced before it returns. See below example

Placeholder for field name.

```php
$validator->setMessage('username', 'required', 'We need your {field}');
```

As per above `{field}` will be replaced by its field name `username`. 

We can also add placeholder for validation rule name.

```php
$validator->setMessage('username', 'required', '{rule} validation failed for {field}');
```

`{rule}` will be replaced by `required` and `{field}` will be replaced by `username`.


We can also print parameter to passed to validation rule starting from index `0`.

Passing placeholder which does not exists will keep placeholder as it is.

##### Validate multiple rule

We can add multiple validation rule for single field. We don't want to only `required` validation rule for each field and but we also want to more validation for fields like username should be at least 3 character long and etc.

Let's take an example of registration form where we can have multiple fields and multiple rules for the field. In a registration form let's talk about username which should have following validation.

1. Username field is required.
2. Username should be at least 3 character long
3. Username length should not be greater than 15 character long.
4. Username should contains only alpha numeric character.

Above rule can be created by following way:

```php
$validator->setValidation([
    'username' => [
        'required',
        'string:min' => 3,
        'string:max' => 15,
        'string:alphanum',
    ]
]);
```

Because `required` and `string:alphanum` does not require any parameter for the rule so we can directly pass rule as value in an array.

Some rule requires parameter like `string:min`, `string:max` where we should inform validator about minimum length and maximum length. This can be done by specifying rule name in key and its parameter in value.

Because in above example rule required only one parameter so we passed parameter directly but if such rule requires more than one parameter we can pass it in array.

As in our example we added rule for min and max length both but validation also has `string:range` rule which can checks if specified field's length is in min and max or not. See below code for `string:range` rule.

```php
$validator->setValidation([
    'username' => [
        'required',
        'string:range' => [3, 15],
        'string:alphanum',
    ]
]);
```

##### Validate CSRF

We just need to instance of `CSRF` to validator and then when we validate request CSRF will be first to check. Once CSRF verified, Validator proceeds for validation of rule.

```php
$csrf = new CSRF('loginForm');
## Change CSRF as per need

$validator->setCSRF($csrf);
```

Please learn CSRF [here](/learningCenter/security/csrf).

##### Validation rules

Nishchay provides various kind validations which are divided by their types and are contained in their own class. These validation can be directly be checked without `Validator`. 

As `Validator` class validate only request parameter, we can Validation rule class to validate data other than request parameter. We only need instantiate validation rule class and then use any of its method. Validation rule class methods returns boolean value only. These classes are belongs to `Nishchay\Validation\Rule` namespace. 

These types are listed below:

| Name | class| Description |
|-----|-----|-----|
| `mixed` | `MixedRule` | This contains validations like `required`, `email` and etc. |
| `string` | `StringRule` | Validations for data type string which includes `min`, `max` and etc |
| `number` | `NumberRule` | Contains validation for data type number |
| `date` | `DateRule` | Contains validations for date and datetime |
| `file` | `FileRule` | Validations for uploaded file |

To use any validation in validator class, it should be used by `{ruleType}:{ruleName}` format.

###### Mixed

For mixed validation, we don't need prefix it with `mixed`. Prefix rule with `mixed` does not creates any issue.

| Name | Description | Example |
|-----|-----|-----| 
| `required` | Checks if input is present and not empty | No parameter |
| `url` | Should be URL | No parameter |
| `email` | Validate email address | No parameter |
| `ip` | Should be IP address | No parameter |
| `ipv4` | Should be IPv4 address | No parameter |
| `ipv6` | Should be IPv5 address | No parameter |
| `mac` | Should be MAC address | No parameter |
| `enum` | Should be in given list | `[['male', 'female', 'other']]` |
| `minCount` | Count of value should at least provided number | `3` |
| `maxCount` | Count of value should not be greater than provided number | `10` |

Below code demonstrate how rule can added for validations with parameter and without parameter.

```php
$validator->setValidation([
    'email' => [
        'required',
        'email' # Rules such as url, ip, mac can used like this
    ],
    'gender' => [
        'enum' => [
            ['male','female','other'] 
        ]
    ],
    'items' => [
        'minCount' => 3
    ]
])
->setMessage('items', 'minCount', 'Expecting at least {0} items');
```

Note how we used `enum`, we are passing array as first element because this rule requires list of allowed item. Because `minCount` need only parameter which can be passed directly instead of in an array. Above code also demonstrate how to add message for rule.

We can also validate directly using its own class.

```php
$mixedRule = new MixedRule();

$gender = 'invalid';
$mixedRule->email($gender, ['male', 'female']);
```

###### String

| Name | Description | Example |
|-----|-----|-----| 
| `stirng:min` | Checks string character length is greater than specified number | 3  |
| `stirng:max` | Checks string character length is less than specified number  | 20 |
| `extact` | String character length must be equals to provided number | 20 |
| `stirng:range` | Checks string character length should be between min and max range | [5,15] |
| `stirng:alpha` | String should contain only alphabates | No parameter |
| `stirng:alphaspace` | String should contain only alphabates and space | No parameter |
| `stirng:alphanum` | String should contain only alphabates and numbers  | No parameter |
| `stirng:alphanumspace` | String should contain only alphabates, numbers and space  | No parameter |
| `stirng:startswith` | Value should start provided needle | foo |
| `stirng:endswith` | Value should ends with provided needle | bar |
| `stirng:lowercase` | Value should be in lowercase | No parameter |
| `stirng:uppercase` | Value should be in uppercase | No parameter |
| `stirng:json` | Value should be in JSON | No parameter |

See below for how string rule can be used along custom message.

```php
$validator->setValidation([
    'username' => [
        'string:min' => 3
    ],
    'name' => [
        'string:range' => [3,20]
    ],
])
->setMessage('name', 'string:range', 'Length of name should be in range {0} and {1}.');
```

Using `StringRule` class. Here method will be without `string:` prefix. Below code will validates if string length is in range.

```php
$stringRule = new StringRule();

# Here first parameter is value and then following parameter are min and max
$stringRule->range('fail', 5, 20);
```

###### Number

Number validation rule contains following validations.

| Name | Description | Example |
|-----|-----|-----| 
| `number` | Value should be number | No parameter  |
| `number:min` | Value should be greater than provided number | 3  |
| `number:max` | Value should be less than provided number | 20  |
| `number:range` | Value should be between provided min and max limit | [3,20]  |
| `number:odd` | Value should be odd | No parameter  |
| `number:even` | Value should be even | No parameter  |
| `number:negative` | Value should be less than zero | No parameter  |
| `number:positive` | value should be greater than zero | No parameter  |

Notice first validation rule, which checks if value is number or not. This rule should be specified without `number:` prefix as its validation rule same as validation rule type.

Example:

```php
$validator->setValidation([
    'age' => [
        'number',
        'number:min' => 18
    ]
])
->setMessage('age', 'number:min', 'Age should be greater than {0}');
```

Above will validates that age should be number and its should be greater than `18`.

Using `NumberRule` class.

```php
$numberRule = new NumberRule();

# Here first parameter is input value and second is required minimum number
$numberRule->min(13, 18);

```

###### Date

Date validation rule contains validations for date format, date range and etc. Date validation requires format validation first and then other validation. See below list of validation.

**NOTE** Validation which requires another date, that date should be in format same as provided in `format` rule.

| Name | Description | Example |
|-----|-----|-----| 
| `date:format` | Validates date is in provided format | 'Y-m-d'  |
| `date:before` | Date should be before provided date. | '2020-01-01'  |
| `date:after` | Date should be after provided date | '2020-01-01'  |
| `date:range` | Date should be between given date range | ['2020-01-01', '2020-01-30']  |


See below for how it can be used in `Validator` class.
```php
$validator->setValidation([
    'passingDate' => [
        'date:format' => 'Y-m-d',
        'date:after' => '2020-01-01'
    ]
])
->setMessage('passingDate', 'date:after', 'Passing should be after {0}');
```

Using `DateRule` class.

```php
$dateRule = new DateRule();

$dateRule->after('2019-12-31', 'date:after', 'Y-m-d');
```

###### File

File validation rule allow us to validate uploaded file only. These validation are contained in `FileRule` validation class.

| Name | Description | Example |
|-----|-----|-----| 
| `file:minSize` | Minimum file size in KB | 200  |
| `file:maxSize` | Maximum file size in KB | 500  |
| `file:minWidth` | Minimum width in pixel. Applicable for image only | 400  |
| `file:maxWidth` | Maximum width in pixel. Applicable for image only | 600  |
| `file:minHeight` | Minimum height in pixel. Applicable for image only | 400  |
| `file:maxHeight` | Maximum height in pixel. Applicable for image only | 600  |
| `file:mime` | List of allowed mime | ['image/jpg', 'image/jpeg']  |


Example:

```php
$validator->setValidation([
    'profilePic' => [
        'file:minSize' => 200,
        'file:mime' => [['image/jpg', 'image/jpeg']]
    ]
])
->setMessage('profilePic', 'file:mime', 'Invalid file for profile picture.');
```

Using `FileRule` class. Because this validation rule is applicable for uploaded file only. We must pass file which we get using `Request::file` method.

```php

$profilePic = Request::file('profilePic');

$fileRule = new FileRule();
$fileRule->mime($profilePic, ['image/jpg', 'image/jpeg']);

```