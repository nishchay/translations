#### Container

Container let us combine group of things of together. This class can have various methods which just returns class name. When we use container and call this method, nishchay creates instnace of returned class and respond as respond as called method. If there is any parameter in class constructor(This is called depdency of class), nishchay resolves this depedency.

We can have as many as we want in our application. Container should be created for group of similar things. Suppose for an example if we create container for Order which can be used of order related implementaiton. As order implementation also involves Login, Cart management, Payment and more. All of these can be combined together in container.

Let's create container for said example.

```php

namespace Application\Container;

/**
* @Container
*/
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

We can use container with the help of `Nishchay::getContainer` which should be called container class name.

```php
$login = Nishchay::getContainer(OrderContainer::class)->getLogin();
```

When we call `Nishchay::getContainer(OrderContainer::class)` it returns singleton instance of container class. Singleton means every time we call conatiner it returns same instance. Once we have instance of container we can call method of container class, this will return instance of returned class name. This is again returns singlton instance of returned class name.

###### Entity
