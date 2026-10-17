#### Dependency Injection

Nishchay Dependency Injection(DI) resolves dependencies of method parameter and property of class. Because this concept is bit complicated, we will take an example to explain how dependency injection work and how we can use it.

Below we have created two class `Account` and `AccountPermission`.

```php
class Account {

    private $accountId;

    private $groupId;

    public function __construct($accountId, $groupId) {
        $this->setAccountId($accountId)
            ->setGroupId($groupId);
    }

    public function setAccountId($accountId) {
        $this->accountId =$accountId;
        return $this;
    }

    public function setGroupId($groupId) {
        $this->groupId =$groupId;
        return $this;
    }

    public function getAccountId() {
        return $this->accountId;
    }

    public function getGroupId() {
        return $this->groupId;
    }
}

class AccountPermission {

    private $account;

    public function __construct(Account $account) {
        $this->account = $account;
    }

    public function getAccount () {
        return $this->account;
    }

    public function getPermissionList () {
        return (new Query)
            ->setTable('AccountPermission')
            ->setCondition([
                'accountId' => $this->getAccount()->getAccountId(),
                'groupId' => $this->getAccount()->getGroupId()
            ])
            ->get();
    }
}
```

Now what we want to do is create an instance of `AccountPermission` permission class, but its constructor requires instance of `Account` class. So Let's create an instance of `Account` class first but this class also requires its dependency called `accountId` and `groupId`.

```php
$account = new Account(121, 10);
```

Now we are done with everything, so let's create and instance of `AccountPermission` then get list of permissions. Our final code will be as shown below:

```php
$account = new Account(121, 10);

$accountPermission = new AccountPermission($account);

$accountPermission->getPermissionList();
```

Whenever we have to create an instance of `AccountPermission`, we also need instance of `Account` where `Account` class also need `accountId` and `groupId`. Here `Account` class dependency of `AccountPermission` class and `accountId` & `groupId` are dependency of `Account` class.

DI makes above things easier by resolving those dependencies.

###### Creating instance

First we need to create an instance of `Nishchay\DI\DI` then using `create` using method we can create instance of any class. We only need to pass class name then DI resolves its dependency.

```php
$di = new \Nishchay\DI\DI();
```

`DI` class first checks `__construct` method of passed class then it starts each parameter of constructor. For parameter which has type hint of class, DI creates an instance of type hinted class after resolving its dependency. If there's further dependency, DI keeps going till end and resolves all dependency wherever is required.

For parameter which does not have type hint, DI looks for its value in resolve list. As in our example `Account` class requires two parameter `accountId` and `groupId`.

Now to create an instant, let's add both of above dependency to resolve list.

```php
$di->set([
    'accountId' => 123,
    'groupId' => 2,
]);
```

After that when we create an instance of `AccountPermission` class. All of its dependency will be resolved by `DI`.

```php
$accountPermission = $di->create(AccountPermission::class);
```

As you can see from above, we do not have create an instance of `Account`. `DI` will create instance of `Account` class and passes it to `AccountPermission` constructor.

##### Resolve list

Before `DI` creates instance of dependent class, it first checks for instance in resolve it. If there's none then `DI` creates an instance of dependent class. As in our example it did for `Account` class.

`DI` checks for dependency in resolve list in following order:

1.  Parameters passed as second argument of create method of DI class.
2.  Class Resolve list
3.  Global resolve list

If parameter does not found in any of above then DI checks parameter is optional or not. For the optional parameter DI uses default value while in case of required parameter instance created only if it has type hint class. If parameter is required and it has no type hint class, then DI throws `Nishchay\Exception\UnableToResolveException` exception.

###### Local resolve list

Create method of DI allows dependencies to be passed via its second argument. Below example demonstrates how we can pass resolve list while creating an instance.

```php
$accountPermission = $di->create(AccountPermission::class, [
    'accountId' => 123,
    'groupId' => 2,
]);
```

##### Class list

Dependency added to `DI` instance using its `set` method is called class resolve list. This makes dependency to be reuse.

```php
// Add single dependency.
$di->set('accountId', 123);
```

We can also set multiple dependencies at once:

```php
$di->set([
    'accountId' => 123,
    'groupId' => 2,
]);
```

##### Global Resolve list

We can add dependency to global scope so that it can be used for multiple instance of DI class.

`Nishchay\DI\DependencyList` have various methods to add, remove and get dependencies. All methods of this class are static. To add dependency use `set` which allow us to add single or multiple dependency at time.

Below example demonstrates how single or multiple dependency are added.

```php
// Add single dependency.
DependencyList::set('accountId', 123);

// Add multiple dependency at a time.
DependencyList::set([
    'accountId' => 123,
    'groupId' => 2,
]);
```

###### Remove dependency

Dependency once added can also be removed by using `remove` method.

```php
$di->remove('accountId');
```

Remove from global resolve list:

```php
DependencyList::remove('accountId');
```

###### Get Dependency

We can get all or specific dependency by its name using `get` method. This methods accepts only one optional argument when passed it returns dependency value of given name. Exception is thrown in case of dependency value does not exist for name. Omitting this name returns all added dependencies.

Fetch all dependencies from class list:

```php
$di->get();
```

Fetch all from global list:

```php
DependencyList::get();
```

Fetch single from class resolve list:

```php
$di->get('accountId');
```

Fetch single from global resolve list:

```php
DependencyList::get('accountId');
```

###### Check if Dependency value exist

DependencyList class also have method called `exist` which can be used to check if dependency value exist of given name. This method returns true or false based on dependency value exist or not.

##### Interface or parent class

DI creates instance of class found in parameter type hint, but sometime this type hint can be interface, abstract class or parent class. In this case we can set what to do. Let's take at below example.

```php
class Uploader {
    public function __construct(UploadInterface $upload) {

    }
}
```

Based on above case, `UploadInterface` can be implemented by various kind of upload method, such method can be `FileUpload`, `S3Upload`. If want `$upload` to be `S3Upload`, we can do that by following ways.

```php
$di->set(UploadInterface::class, S3Upload::class);
$di = $di->create(Uploader::class);
```

Now when DI resolves dependency it will create instance of `S3Upload` in the case wherever it finds `UploadInterface` interface.

This can be changed later if want `FileUpload` instead of `S3Upload`:

```php
$di->set(UploadInterface::class, FileUpload::class);
```

##### Single instance

Every time we call `create` method of DI class, it creates new instance and returns it. But we can prevent it to be created every time and create method returns instance which was already created. Third parameter of `create` method tells DI to store instance in `InstanceList` if it was not been created before.

```php
$di = new \Nishchay\DI\DI();
$accountPermission = $di->create(AccountPermission::class, [], true);

# Reusing already created instance
$accountPermission2 = $di->create(AccountPermission::class);
```

Here $accountPermission and $accountPermission2 will be same as first call with AccountPermission will create instance and it will store created instance in `InstanceList` because we have passed third parameter `true`. For the second we are not passing second and third parameter as its not required, passing it will not make sense. Second call will returns the same instance which was created using first call on AccountPermission class.

###### Remove single instance

Instance which has been saved to `InstanceList` can also be removed, we only need pass class name. If there's instance created for class exists in list, then it removes instance form list and returns `true` otherwise it returns `false`.

```php
$di->removeInstance(AccountPermission::class);
```

##### Invoke method

Apart form creating instance of class DI also provides method to invoke method class. We can invoke method of any instance even if it was not created using DI.

```php
$di->invoke($instance, 'methodName');
```

Invoking method must be public. Invoking method which is not accessible results in an exception. Before calling method, DI first resolve all parameter of method.

We can also pass resolve list in third argument.

##### Resolve class property

Class properties are resolved by assigning its value by looking property name if exist in class resolve list or global dependency list.

###### All properties

Calling `resolveAllProperty` method resolves each properties of class whether its private, protected or public. This method excepts two argument first being instance class whose properties needed to be resolved. Second ignore list, if passed property name exist ignore list are not resolved.

```php
$di->resolveAllProperty($instance)
```

###### Ignore property

Property can ignore from being by two ways. By passing second argument in `resolveAllProperty` as list of properties. Or we can add properties to ignore list by using `setIgnored`. Properties once added to ignore list can be removed by using `removeIgnored`. As this list stored in class, this list applies to each call resolveAllProperty.

```php
//Set ignore property.
$di->setIgnored('firstName');

//Remove ignored property from list.
$di->removeIgnored('firstName');
```

###### Resolve specific property

To specific property only of class use `resolveProperty`. This method accepts two arguments first being instance of class and second name property which needs to be resolve.

```php
    $di->resolveProperty($instance, 'firstName')
```
