#### Nishchay: decide how your app is built, then build it

`Nishchay` is an open structure PHP framework, released under the New BSD license in 2020 and written by `Bhavik Patel`. Most frameworks hand you a folder layout and ask you to live with it. Nishchay lets you **define the structure of your application yourself**, then enforces it, so the code stays organised as it grows.

It is written for PHP 8 and built on native **attributes**: routes, services, entities and validation are declared on the class or method they belong to. You can use it in personal and commercial projects.

The name means _decision_. It comes from the idea that to do anything, we only need to decide and start working on it.

##### A look at the code

A JSON endpoint is one method with a few attributes. There is no separate routes file to keep in sync, and the `{userId}` segment is passed to the method parameter of the same name.

```php
use Nishchay\Attributes\Controller\Controller;
use Nishchay\Attributes\Controller\Method\{Route, Response, Service, Placeholder};

#[Controller]
class UserController
{
    #[Service]
    #[Route(path: 'user/{userId}')]
    #[Placeholder(['userId' => 'int'])]
    #[Response(type: 'json')]
    public function profile(int $userId)
    {
        return ['name' => 'Bhavik Patel', 'accountType' => 'Developer'];
    }
}
```

A database table is a class, and encrypting a column is one parameter.

```php
use Nishchay\Attributes\Entity\Entity;
use Nishchay\Attributes\Entity\Property\{Identity, DataType};

#[Entity(name: 'user')]
class User
{
    #[Identity]
    #[DataType(type: 'int', readOnly: true)]
    public $userId;

    #[DataType(type: 'string', length: 100, required: true, encrypt: true)]
    public $email;
}
```

With encryption configured, `email` is stored encrypted and decrypted when you read it back. You do not write the query: Nishchay builds it from the entity for the database you use.

##### Why choose Nishchay

###### 1. Your structure, not ours

The layout of your application lives in one XML file, `settings/configuration/structure.xml`. Choose MVC, HMVC or a structure of your own. Nishchay then checks controllers, entities, events and views against it, so the pattern you chose does not drift as the team and the codebase grow. [Read about open structure](/learningCenter/openStructure).

###### 2. Everything is declared where it is used

Routes, placeholders, response types, services, entities, properties, events and validation all use PHP 8 attributes. To learn how an endpoint behaves, you read its method. The consistency also makes the code easy for editors and AI coding tools to read and write.

###### 3. A lot is already built

You do not need to assemble packages for the everyday parts of an application. These are part of the framework:

| Area | What you get |
| ---- | ------------ |
| Routing | SEO-friendly paths, RESTful methods, placeholders, scopes, and route visibility by time, user agent, IP address or callback. |
| Web services | JSON and XML responses, **field demand** so consumers ask for only the fields they need, token-protected services, and your own response formats. |
| Data | Entity classes, relations between entities, events before and after insert, update and delete, an entity query builder, query caching and a database manager for creating and altering tables. |
| Security | Request validation and a form builder, CSRF protection, field encryption, captcha and OAuth2 support. |
| Sessions | Read-limited, next-request, scoped and conditional sessions, stored in files, a database or cache. |
| Maintenance mode | Schedule it by browser version, daily time range, date range or callback, and still allow chosen routes through. |
| Console | Generate controllers and entities (empty, CRUD or from templates), list routes and entities, and run routes from the command line. |
| Everything else | Dependency injection, a container, localization, mail, logging, caching and exception handlers. |

###### 4. Built around a few clear ideas

A controller, a route, an entity and a structure are the core concepts, and each is documented as its own topic. You can learn them one at a time and start building before you have read everything.

##### Is it right for you?

Nishchay is a good fit when:

- You want a codebase whose structure you control and can keep consistent.
- You like attributes and want routes, services and entities declared next to the code.
- You are building websites or REST APIs and want a lot included without choosing between many packages.
- You would rather learn one coherent framework than assemble an ecosystem.

Be aware that Nishchay is a young framework with a small community. If you rely on a very large catalogue of third-party integrations, that will matter. If you want to help close that gap, [contribute](/contribute).

##### Start in minutes

```bash
composer create-project nishchay/nishchay nishchayApp
```

1. Open [Get Started](/learningCenter/getStarted), choose your operating system and what you want to build, and it gives you an installation guide and a reading path. You need PHP 8.3 or newer and Composer.
2. Create your [first controller and route](/learningCenter/getStarted/web), or a [REST API](/learningCenter/getStarted/restApi).
3. Learn how the [request life cycle](/learningCenter/lifeCycle) works, then explore [entities](/learningCenter/data/entity), [validation](/learningCenter/security/validation) and [web services](/learningCenter/service/webService).

##### Where next

- **New to the framework:** [Get Started](/learningCenter/getStarted)
- **Want to see how it is organised:** [Open structure](/learningCenter/openStructure) and [application patterns](/learningCenter/applicationPattern/nishchay)
- **Building an API:** [Web service](/learningCenter/service/webService)
- **Working with data:** [Entity manager](/learningCenter/data/entity)
- **Everything else:** the documentation menu on the left.
