#### Container

Container let us combine group of things of together. This class can have various methods which just returns class name. When we use container and call this method, nishchay creates instance of returned class and returns it. If there is any parameter in class constructor(This is called depdency of class), nishchay resolves this dependency.

We can have any number of containers in our application. Container should be created for group of similar things. Suppose for an example if we create container for Order which can be used of order related implementation. As order implementation also involves Login, Cart management, Payment and more. All of these can be combined together in container.

Let's create container for said example.

```php
use Nishchay\Attributes\Container\Container;

namespace Application\Container;

#[Container]
class OrderContainer {
    public function getLogin() {
        return \Application\Components\Login::class;
    }

    public function getCart() {
        return \Application\Components\Cart::class;
    }

    public function getPayment() {
        return \Application\Component\Payment::class;
    }
}

```

##### How can we use

###### Nishchay::getContainer

We can use container with the help of `Nishchay::getContainer` which should be called with container class name.

```php
$login = Nishchay::getContainer(OrderContainer::class)->getLogin();
```

When we call `Nishchay::getContainer(OrderContainer::class)` it returns singleton instance of container class, for above case its `OrderContainer`. Singleton means every time we call container it returns same instance. Once we have instance of container we can call method of container class. Calling method returns instance of class which was returned by method of container class.

As in above example calling `getLogin` method will return instance of `Application\Components\Login`. This magic happens within nishchay container processor.

###### Facade

When we declare `FACADE` constant in container class, value of this constant can be use as class. We can directly use class name and then call method of container statically.

Let's modify our above container by adding `FACADE` constant.

```php
use Nishchay\Attributes\Container\Container;

namespace Application\Container;

#[Container]
class OrderContainer {
    const FACADE = 'OrderContainer';
...
```

When we declare `FACADE` constant in container class then nishchay will create class as specified this constant. This class is special class as it allows calling container method statically.

Let's call one of method of above container class.

```php
$login = OrderContainer::getLogin();
```

NOTE: Class name specified in facade constant should not be already exists. If you specify class name which already exists, then it will result in an exception.

###### Entity

When method returns class name of entity, then container returns instance of `EntityManager` for entity class.

```php
public function getUser() {
    return \Application\Entities\User::class;
}
```

Calling `Nishchay::getContainer(OrderContainer::class)->getUser()` will return instance of `EntityManger` for `User` entity which will be similar to following:

```php
$userEntity = new EntityManager(User::class);
```

###### Return instance of instance

Sometime we might need some calculation or preparation for method parameter before creating an instance of class, in this case we can still use container. We can return of instance of class rather its class name if its required.

```php
public function getPayment() {
    $token = ThirdPartyPaymentGateway::createToken();

    return new Payment($token);
}
```

If we call above method multiple times through container, it will still return same instance. This is because on the first call, returned instance is stored in container registry and then calling again returns instance from registry

##### Get new instance

As container method always returns same instance every time, we can still request new instance if we need it. We just need to pass `true` while calling container method, below code always returns new instance.

```php
$userEntity = Nishchay::getContainer(OrderContainer::class)->getUser(true);
```

###### Dependency

Some class requires parameter to create an instance, these parameters can be scaler type or instance of class. All these parameters are known as dependency of class. When we use container, we don't need to pass these parameter as this will be resolved by container when it creates instance of returned class name.

In container we should declare `resolveList` property which will have list of values for the dependency required by any class within container. When container creates instance of class it will first look at this property for parameter is exists or not.

```php
class OrderContainer  {
    private $resolveList = [
        'class' => \Application\Entities\Payment:class
    ];

    public function getPayment() {
        return \Application\Component\Payment::class;
    }
}
```

Payment class

```php
namespace Application\Component;
class Payment {

    private $paymentEntity;

    public function __construct(EntityManager $entity) {
        $this->paymentEntity = $entity;
    }
}
```

Now when we call `getPayment` method on container, it will return instance of `Application\Component\Payment` class. But how container creates instance of this class.

When container tries to create instance of returned class name, it will first resolve dependency of that class. In above case container `Application\Component\Payment` requires instance of `EntityManager` so container will first check if any value exists with name `entity` in `resolveList` property, if it does not exists there then it create instance of `EntityManager` and then pass it to `Application\Component\Payment` class.

But again `EntityManager` requires class name to be passed. Now again container look for value with name `class` in container `resolveList` property.

As we specified `class` in `resolveList`, container will take it and passes it to `EntityManager` to create an instance of it. Container can resolve any level of nested dependency.

###### Dependency parameter

While calling container method we can also pass parameter so container can look for dependency in argument passed. In this case first priority will be argument passed then `resolveList` will be considered.

```php
$userEntity = Nishchay::getContainer(OrderContainer::class)->getUser(false, ['dependentName' => 'value']);
```

Here we are passing first parameter as `false` because we want singleton instance. On further call we can omit above parameter in `getUser` method, because it will return instance from registry only.

##### Container rule

1. Container method must start with `get`, method which does not start with `get` are ignored. Container does not allow calling method doest not start with `get`.
2. Method must be public.
3. Method must not be static.
4. Method name must not start with underscore.
