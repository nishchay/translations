#### Form Builder

Nishchay provides class based approach to create form including its validations. In form class, method represents form fields. Form class also protects against CSRF(Cross site request forgery). Form class must extends `Nishchay\Form\Form` to have all features of form building, validations and csrf.

`Form` class have various method to create input field for the form. In form class method which follows `get{FieldName}` are considered as form fields. Here `{Fieldname}` must have first character in upperCase. Inside form field method, we can use any of following methods to create form field for the form.

| Method         | Description                               | Class         |
| -------------- | ----------------------------------------- | ------------- |
| newButton      | To create button                          | `Button`      |
| newInput       | To create input field like text, password | `Input`       |
| newInputChoice | To create radio or checkbox button        | `InputChoice` |
| newSelect      | To create select field                    | `Select`      |
| newTextArea    | To create textarea                        | `TextArea`    |

Method returns instance of form field class is mentioned in class column. All of Field class are part of `Nishchay\Form\Field\Type\` namespace.

###### Form class

Let's first create form class then will move to its explanation.

```php
use Nishchay\Form\Form;
use Nishchay\Http\Request\Request;
use Nishchay\Attributes\Form\Form as FormAttribute;

/**
* Login form class
*/
#[FormAttribute]
class RegisterForm extends Form {

    /**
    * Just calls parent construct method for setting form name and its HTTP method.
    */
    public function __construct()
    {
        parent::__construct('register-form', Request::POST);
    }
}
```

Constructor of `Form` class requires two parameter, form name and HTTP method. Form name is used for CSRF token to be saved in Session and it should be unique throughout application.

##### Printing form

To print form we first we need create an instance of form than we can use `startForm` method to print start tag of form and `endForm` to print end tag of form.

###### Form start tag

```php
$form = new RegisterForm();

echo $form->startForm();
```

Above code will print below tag:

```html
<form method="POST" action="">
  <input type="hidden" name="csrf" value="{CSRF_TOKEN}" />
</form>
```

By default form action is current route. We can action using attributes. Learn more about attribute in `Attribute` section.

###### Form end tag

```php
echo $form->endForm();
```

Which will prints:

```html
</form>
```

##### Attributes

`Form` and Form field classes contains `setAttribute` method to set attribute for form and form field. Using which can set attributes for form like action, onsubmit event and etc. Same thing applies for form fields.

In `Form start tag` section we have printed form which have action to current route. Let's change action attribute of form. Setting attributes of form should be done in `__construct` method only. For the form field it should be in its method.

```php

public function __construct()
{
    parent::__construct('register-form', Request::POST);
    $this->setAttribute('action', '/services/accountService/register')
            ->setAttribute('onsubmit', 'return validateRegisterForm(this)');
}
```

Now if we print start tag of form, it will print below HTML code:

```html
<form
  method="POST"
  action="/services/accountService/register"
  onsubmit="return validateRegisterForm(this)"
></form>
```

##### Form fields

To better understand each form fields we are taking an example for registration. We will start with creating text than we will keep improving our Form class.

Each form field method except `newTextarea` and `newSelect` requires two parameter, first one for field name and second for the input type. Each method in form class must follow `get{FieldName}` format for the field and method must return form field.

##### Input field

Using `newInput` we can create form fields like text, password and etc. This method does not allow creating radio and checkbox field which have its separate method `newInputChoice`.

###### Text field

```php
use Nishchay\Form\Form;
use Nishchay\Attributes\Form\Form as FormAttribute;

#[FormAttribute]
class RegisterForm extends Form {
    # Constructor code

    public function getEmail() {
        return $this->newInput('email', 'text');
    }
}
```

This is it, we have create form field for RegisterForm. Now let print this input field.

```php
echo $form->startForm();
echo $form-->getEmail();
echo $form->endForm();
```

HTML output of above code:

```html
<form method="POST" action="">
  <input name="email" type="text" />
</form>
```

Similarly we can also create password field.

```php
public function getPassword() {
    return $this->newInput('password', 'password');
}
```

###### File

To create file form field, pass input type as file while creating form field.

```php
public function getProfilePic() {
    return $this->newInput('profilePic', 'file');
}
```

###### Radio

Radio and checkbox fields are created using `newInputChoice` method. Below code demonstrate how radio button for the `gender` option can be created.

```php
use Nishchay\Form\Form;
use Nishchay\Attributes\Form\Form as FormAttribute;

#[FormAttribute]
class RegisterForm extends Form {

    # Other fields

    public function getGender() {
        return $this->newInputChoice('gender', 'radio');
    }
}
```

Method `getGender` won't print anything if we call it, because we haven't added any choices to it. We can add choices using `setChoices` method. All choices should be passed in one call as an array where key presents choice value and value presents its HTML value.

```php
use Nishchay\Form\Form;
use Nishchay\Attributes\Form\Form as FormAttribute;

#[FormAttribute]
class RegisterForm extends Form {

    # Other fields

    public function getGender() {
        return $this->newInputChoice('gender', 'radio')
                        ->setChoices([
                            'male' => 'Male',
                            'female' => 'Female'
                ]);
    }
}
```

Now just call `getGender` and it will print below HTML code:

```html
<input name="gender" type="radio" value="male" /> Male
<input name="gender" type="radio" value="female" /> Female
```

Calling `getGender` prints all choices but in some case we might need to print one choice at a time. We can print single choice `getChoice` method.

Here first parameter should be choice name and second is for printing its HTML value along with it.

```php
echo $form->getGender()->getChoice('male');
```

Above will print only one choice:

```html
<input name="gender" type="radio" value="male" Male />
```

We can also print choice without its HTML value

```php
echo $form->getGender()->getChoice('male', true);
```

To get only HTML value of choice:

```php
echo $form->getGender()->getChoiceHTML('male');
```

###### Checkbox

`Checkbox` field can be created just like `radio` button, we just have to pass `checkbox` type while creating field.

```php

public function getIsTermsAccepted() {
    return $this->newInputChoice('isTermsAccepted', 'checkbox')
                        ->setChoices([
                            '1' => 'Please accept terms'
                ]);
}

```

HTML output of above field:

```html
<input name="isTermsAccepted" type="checkbox" value="1" /> Please accept terms
```

###### Select

Select field is created using `newSelect` method.

```php
public function getGender()
{
    return $this->newSelect('gender')
                    ->setChoices([
                        'male' => 'Male',
                        'female' => 'Female'
            ]);
}
```

Printing Select field

```php
echo $form->getGender();
```

Above will print below HTML code.

```html
<select name="gender">
  <option value="male">Male</option>
  <option value="female">Female</option>
</select>
```

###### Textarea

Using `newTextarea` we can create textarea form field, see below code:

```php
public function getAboutYou()
{
    return $this->newTextArea('aboutYou');
}
```

Printing above form field:

```php
$form->getAboutYou();
```

Above code will output HTML code as shown below:

```html
<textarea name="aboutYou"></textarea>
```

###### Button

Using `newButton` we can create submit or normal button. Below code demonstrate how submit can be created.

```php
public function getSubmit()
{
    return $this->newButton('submit', 'Submit')
            ->setValue('Submit');
}
```

Printing button:

```php
$form->getSubmit();
```

HTML form of above button

```html
<button name="submit" type="submit">Submit</button>
```

###### Form field value

All of form fields have `setValue` using which we can set value of field:

```php
$form->getEmail()->setValue('demo@example.com');
```

##### CSRF[time:120]

Form class has CSRF enabled by default. Upon creating an instance of Form class it also initializes CSRF for the form and saves CSRF into session. When we print form using `startForm` it also prints CSRF token in input hidden field so that when users submit form, server receives same token and validator verify received token with token saved in session for the submitted form. If there's mismatch in token then `Nishchay\Exception\BadRequestException` exception is thrown.

Each form has its name and CSRF token is saved in session with `csrf_{formName}` format. This is why we should use unique name for each form in an application.

By default CSRF configured to be received in request method same as form and with`csrf` parameter name. This can be changed using `getCSRF` method which returns instance of `CSRF`. See below code:

```php
use Nishchay\Form\Form;
use Nishchay\Attributes\Form\Form as FormAttribute;

#[FormAttribute]
class RegisterForm extends Form {

    public function __construct()
    {
        parent::__construct('register-form', Request::POST);

        $this->getCSRF()
                ->setName('token');
    }
}
```

Above code just changes csrf name. Learn [here](/learningCenter/security/CSRF) for more about `CSRF` for changing its configuration.

Whenever we call `validate` method to validate form, validator first verifies CSRF. Once CSRF has been verified CSRF proceeds for validation of form fields.

**NOTE** There's way to disable CSRF for the form but we recommend not to do that unless form is being used in web services. Below code shows how we can disable CSRF for the form

```php
use Nishchay\Form\Form;
use Nishchay\Attributes\Form\Form as FormAttribute;

#[FormAttribute]
class RegisterForm extends Form {

    public function __construct()
    {
        parent::__construct('register-form', Request::POST);

        $this->removeCSRF(true);
    }
}
```

If we pass `false` instead of `true` it enables CSRF again.

##### Form field validation rules

We can add all kind of validation to form fields. Please learn more various validations [here](/learningCenter/security/validation).

###### Make field required

```php
public function getEmail()
{
    return $this->newInput('email', 'text')
            ->isRequired();
}
```

To make optional again, pass `false` to `isRequired` method.

###### Set validations

We can set single or multiple validation for the field at time. If we want to set multiple validations in one call pass validations rule in an array. Below code demonstrate how we can add following validation rule to form field.

1. Field is required
2. Value should contain only alphabets.
3. Value character should be between 3 and 20.

```php
public function getFullname()
{
    return $this->newInput('fullName', 'text')
                    ->isRequired(true)
                    ->setValidation('string:alpha')
                    ->setValidation('string:min', 3)
                    ->setValidation('string:max', 20);
}
```

Using array:

```php
public function getFullname()
{
    return $this->newInput('fullName', 'text')
                    ->isRequired(true)
                    ->setValidation([
                        'string:alpha',
                        'string:min' => 3,
                        'string:max' => 20
                    ]);
}
```

Setting validation for the field is same as setting validation using `Validator` class. In `Validator` we also need to pass field name but for the form field we do not have to do that because we call set validation on form field.

###### Set message

We can set single or multiple messages in one call. To set multiple message, pass an array where key presents rule name and its value is validation message.

```php
public function getFullname()
{
    return $this->newInput('fullName', 'text')
                    ->isRequired(true)
                    ->setValidation([
                        'string:alpha',
                        'string:min' => 3,
                        'string:max' => 20
                    ])
                    ->setMessage([
                        'required' => 'Please enter Full name',
                        'string:alpha' => 'Full name should contain only alphabet'
                    ]);
}
```

If we don't set validation message, Validator uses default error message.

##### Validating form

Once form is submitted, form validated by calling `validate` method. This method returns `true` if validations are passed otherwise it returns `false`. If there's no validation done it returns `null`.

Before validating form, validator first verifies CSRF. If there's CSRF verification fails, `validate` method throws `Nishchay\Exception\BadRequestException` exception.

```php
if ($form->validate() === false) {
    return $form->getErrors();
}
```

###### Validation error

If form validation fails, we can get all errors by using `getErrors` method.

```php
$form->getErrors();
```

We can also fetch error message of specific field by passing its field name.

```php
return $form->getErrors($form->getEmail()->getName());
```

##### Fetch request value

To fetch request value of any field call `getRequest` method on the form field. This returns `false`, if request parameter does not present for the field.

```php
echo $form->getEmail()->getRequest();
```

This method returns request parameter value for the field based on HTTP method set for the form. As an example, if we set HTTP method `POST` for the form, `getRequest` returns parameter value from `POST` body only.

If form field is file, it checks for file in request body. If there is file uploaded then it returns instance of `RequestFile`.
