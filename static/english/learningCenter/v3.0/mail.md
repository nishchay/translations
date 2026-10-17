#### Mail

Nishchay mail is wrapper around swiftmailer. This makes swiftmailer feature more easy to use. Mail configurations are contained in `settings/configuration/mail.php` file where we can define list of mail configuration.

##### Configuration

Inside `config` section we can define list of mail configuration to be used within application.

```php
'main' => [
    'host' => 'localhost',
    'port' => 1025,
    'encryption' => null,
    'credential' => [
        'username' => '',
        'password' => ''
    ],
    'from' => [
        'test@example.com' => 'Bhavik Patel'
    ]
]
```

Below table explains each config:

| Name | Description |
|-----|-----|
| host | SMTP host |
| port | Port number |
| encryption | Encryption type like tls, ssl. |
| credential.username | Username for authentication. Leave it blank if not needed. |
| credential.password | Password for authentication. Leave it blank if not needed. |
| from | Default email address from which email should be sent. |


Which config to be used by default is defined using `default` setting.

##### How to use

Use `Nishchay::getMail` method to get an instance for the mail. This method accepts optional argument, which should be name of configuration. If we don't pass then instance for default configuration is returned.

Below example demonstrates how we can send simple email:

```php
Nishchay::getMail()
        ->setTo(['test@example.com' => 'Test Name'])
        ->setSubject('My First Email')
        ->setBody('This is my first email content')
        ->send()
```

##### Recipient

Use `setTo` method to set recipient of an email. We can pass single or email address.

```php
Nishchay::getMail()
        ->setTo(['test@example.com' => 'Test Name']);
```

This will sets email to be sent to `test@example.com`. Below code also works if we don't set name for the `to` address:

```php
Nishchay::getMail()
        ->setTo('test@example.com');
```


Just like `setTo`, we can also set CC and BCC using `setCc` and `setBcc` method.


##### Subject

To set email subject use `setSubject` method.

```php
Nishchay::getEmail()
        ->setSubject('Email Subject');
```

##### Write email content

We can set email body content using `setBody` method which accepts following arguments:

| Name | Description |
|-----|-----|
| content | Content of the email |
| contentType | Content type of email content like `text/plan` or `text/html` |
| Charset | Charset like `UTF-8`.  |

To send plain text email, just pass body content only in first argument. For the HTML content pass `text/html` in the second argument.

##### Email content from view

We can also use view for the email content. We only need to `setBodyFromView` method which accepts two argument, first one for the view name and second for the charset.

See below code how we can send email using content from email.

```php
Nishchay::getEmail()
        ->setBodyFromView('mail/firstEmail');
```

We don't have put extension in view name as we do in route. We can use `html`, `php` or `twig` file. To pass data to view use `Nishchay\Http\Request\RequestStore` class. Learn more about Request store [here](/learningCenter/request/requestStore).

##### Embed image in content

Rather than using url in image, we advise to use embedded image in content. Method `embed` helps us embedding image in content. This method returns embedded id for the image path passed to it. Below example demonstrate how we can we use it.

Let's first get an embedded id for the image:

```php
$imageId = $Nishchay::getEmail()->embed(RESOURCES . 'images/nishchay.png');
```

Then pass it to view using `RequestStore`:

```php
RequestStore::set('imageId', $imageId);
```

Use it in view:

```html
<img src="{{ imageId|raw }}" alt="Nishchay Logo"/>
```

##### Attachment

We can attach any file to be sent as an attachment. We simply need to pass path to file and its filename. Although file name is optional here. We recommend file name to be pass. See below example how we can attach a file:

```php
Nishchay::getEmail()
        ->addAttchment('/path/to/file.doc', 'Document');
```

Once attachment has been attached to email, it can be also be removed:

```php
Nishchay::getEmail()
        ->removeAttachment('/path/to/file.doc')
```

This method throws `Nishchay\Exception\ApplicationException` exception if path passed was not attached to an email.


##### Send

Once we done with preparing an email, email can send by using `send` method. This method returns number of email sent successfully.

```php
Nishchay::getMail()
        ->setTo([
            'test@example.com' => 'Test Name',
            'test2@example.com' => 'Test2 Name'
        ])
        ->setSubject('My First Email')
        ->setBody('This is my first email content')
        ->send()
```


##### Priority

To set priority for the email, use `setPriority` method. Which accepts priority number starting from `1 (Highest)` to `5 (Lowest)`.

```php
Nishchay::getEmail()
        ->setPriority(3);
```

Priority number and its levels are show below:

| Number | Description |
|-----|-----|
| 1 | Highest |
| 2 | High |
| 3 | Normal |
| 4 | Low |
| 5 | Lowest |