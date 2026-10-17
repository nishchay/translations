#### Get started: Your first resource

In this tutorial you generate a small **tasks** API, call it, and then change it. It takes about ten minutes, and it shows how the pieces of Nishchay fit together. The same guide is on the home page of every new Nishchay application, with buttons that send the requests for you.

##### Before you start

You need a Nishchay application that runs on your machine and a database it can connect to.

If you do not have an application yet, open [Get Started](/learningCenter/getStarted), choose your system, and follow the installation guide it links to. It needs **PHP 8.3 or newer** and **Composer**.

The database settings are in `settings/configuration/database.php`. Import `db.sql`, which is in the project folder, to create the `NishchayDB` database. The home page of your application tells you whether it can connect.

To run the application while you work, start PHP's built-in server from the `public` folder of your application:

```bash
cd nishchayApp/public
php -S localhost:8000
```

Leave it running. Every address below uses `http://localhost:8000`. Run the `php nishchay` commands from the project folder, in another terminal.

##### Generate the resource

One command creates an entity, its database table, a form and a controller:

```bash
php nishchay prototype -generate -new
```

It asks a few questions. Answer like this:

| When it asks | Type |
| ------------ | ---- |
| Enter entity name | `Task` |
| Enter property name (the first one becomes the identity) | `taskId` |
| Enter property name, then Select Data type | `title`, then `4` (string) |
| Enter property name, then Select Data type | `isDone`, then `9` (boolean) |
| Enter property name | `q` (to stop) |
| Do you want to create table in DB? | `y` |
| Where do you want to create | `1` (`Application\Controllers`) |
| Enter route name | `tasks` |

The data type numbers come from the list the command prints: 1 int, 2 float, 3 double, 4 string, 5 date, 6 datetime, 7 array, 8 mixed, 9 boolean.

When it finishes you have three new files and a `Task` table in your database:

| File | What it does |
| ---- | ------------ |
| `Application/Entities/Task.php` | Maps the class to the table. |
| `Application/Forms/Task.php` | Validates what clients send. |
| `Application/Controllers/Task.php` | The routes under `/tasks`. |

##### Call your API

Create a task:

```bash
curl -X POST http://localhost:8000/tasks -d "title=Read the docs" -d "isDone=false"
```

```json
{"taskId":1}
```

List the tasks:

```bash
curl http://localhost:8000/tasks
```

```json
[{"taskId":1,"title":"Read the docs","isDone":false}]
```

Get one task, mark it as done, and delete it:

```bash
curl http://localhost:8000/tasks/1
curl -X PUT http://localhost:8000/tasks/1 -d "title=Read the docs" -d "isDone=true"
curl -X DELETE http://localhost:8000/tasks/1
```

```json
{"taskId":1,"title":"Read the docs","isDone":false}
{"message":"Record has been updated"}
{"message":"Record has been removed"}
```

Asking for a task that does not exist answers with status `400` and `{"error":"Record does not exists.","type":"BadRequestException"}`.

A boolean can be sent as `true`, `false`, `1` or `0`.

###### Paging

The list returns 10 records by default. Pass `limit` to get more, between 10 and 50, and `offset` to move through the records:

```bash
curl "http://localhost:8000/tasks?limit=20&offset=20"
```

##### How it fits together

Open the three generated files. Each one is short.

The **entity** is a plain class. Attributes tell Nishchay how it maps to the table.

```php
#[Entity(name: 'this.base')]
class Task
{

    #[Identity]
    #[DataType(type: 'int', readOnly: true)]
    public $taskId;

    #[DataType(type: 'string')]
    public $title;

    #[DataType(type: 'boolean')]
    public $isDone;

}
```

`this.base` means the table is named after the class. Read more in [entities](/learningCenter/data/entity).

The **form** has one method for each field the client may send. This is where validation rules go.

```php
public function getTitle()
{
    return $this->newInput('title', 'text');
}
```

The **controller** declares its routes with the `Route` attribute. The generated routes call the built in `Crud` prototype, which does the work with the entity and the form. You can replace any of them with code of your own.

```php
#[Route(path: '/', type: 'GET')]
#[Response(type: 'json')]
public function index()
{
    return $this->getPrototype()
                    ->get();
}
```

None of this is fixed. Folders such as `Controllers`, `Entities` and `Forms` are a recommendation, and `settings/configuration/structure.xml` describes the layout your application uses. Read more in [open structure](/learningCenter/openStructure).

##### Make it yours

Make the title required. In `Application/Forms/Task.php`, add `isRequired()` to the title field:

```php
public function getTitle()
{
    return $this->newInput('title', 'text')
                    ->isRequired();
}
```

Create a task without a title:

```bash
curl -X POST http://localhost:8000/tasks -d "isDone=false"
```

The API answers with status `400`:

```json
{"errors":{"title":"title is required."}}
```

Other rules work the same way, for example a maximum length or a date format. See [validation](/learningCenter/security/validation).

##### Start from a table you already have

If the table exists, generate the same three files from it:

```bash
php nishchay prototype -generate -table Task
```

It asks where to create the controller and for the route name. Nishchay reads one column that you do not define yourself, `extraProperty`, so a table you create by hand needs it:

```sql
ALTER TABLE Task ADD extraProperty TEXT DEFAULT NULL;
```

Tables created by `prototype -generate -new` already have it. See [console commands](/learningCenter/consoleCommand) for everything the generators can do.

##### Keep exploring

You have the whole loop: an entity, a form, a controller and a working API. [Get Started](/learningCenter/getStarted) lists what to read next for the kind of application you want to build.
