#### Prototype
Prototype is already implemented task which can be configured to your code. To use prototype we just need provide it entity and form class. Apart from this it also provides various other configuration and callback method which executes before or after certain actions. This configuration and callback differs for each prototype. Nishchay provides following prototypes:

1. Auth
2. Register
3. CRUD

Prototype must be used with entity. It does not connect directly to table, it must be through entity only.

##### Auth : Account
This prototype allows authorizing(login) user within an application. This prototype is part of `AccountPrototype`. Below code demonstrates how it can be used.

```php
use Nishchay\Prototype\AccountPrototype;
use Application\Forms\Login;
use Application\Entities\User;

class AccountController{

    /**
    * @Route(path='account/auth',type=POST)
    */
    public function auth(AccountPrototype $account) {
        $response = $account->getAuth(User::class)
                    ->setForm(Login::class)
                    ->execute();
    }
}
```

##### Register : Account

##### CRUD
