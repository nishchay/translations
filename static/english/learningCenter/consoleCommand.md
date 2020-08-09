#### Console Command

Console commands can be used to create controller, route and entities and it can also be used to view application information such as list of controllers, routes, entities and more. 

Commands can also be used for creating CRON job, all GET routes are executable from console commands.

Nishchay console commands are executed by `php nishchay {follow up commands}`. Executing `php nishchay` prints list of supported commands and its description.


##### Command Help

Just execute `php nishchay` without any follow up command and help for console command will be printed. This will print all supported along with how each command can used with one or more optional parameter.

##### Framework version

Framework version is displayed using `php nishchay -v` command. Here `-v` also has two more alias, `v` and `-version`. Once you execute command below information will be printed:

*   Version number
*   Version name

##### Route

`route` command is used to find all routes defined in application. This list can be filtered based on its controller, context and regex pattern. All route with HTTP method GET allowed can be executed by this command.

Using `route` we can find below information related to route:

*   Path as defined in `@Route` annotation.
*   HTTP methods for which route needs to be executed.
*   Route's controller class name
*   Route's method name

###### List all routes

To list all routes in an application execute route command without any parameter.
```
php nishchay route
```
###### Routes belongs to specific controller

Lists of all routes defined in specific controller.

```
php nishchay route -controller {controllerClass}
```

Here controllerClass must be full class path. As class path contains \ (slash), classPath must be passed in single or double quote. If you pass class which is not controller or it does not exists then command will gives an error.

###### Route information by its path name

We can also view route information by its path name as defined `@Route` annotation.
```
php nishchay route -name {pathName}
```
###### Route information by url

To route information by any URL, just pass URL after `php nishchay route` This is helpful when you have large application or starting coding already developed application for the first time as it will gives all information which is required to locate route.
```
php nishchay route {path}
```
###### Execute route

To execute route we only need to pass url and then `-run`. 

**Note:** Only route which supports GET request can be executed using route command.

```
php nishchay route {path} -run
```

###### List route by regular expression

Upon passing regular expression. This regular expression will be run for each route path name and it will list all routes which matches regular expression.
```
php nishchay route -match {RegexPattern}
```
###### Event defined for route

We can also find events which can be execute before or after route. This will list events defined for route, route's scope (if it has been defined on route) and route's context.
```
php nishchay route {path} -event
```
###### Error handler defined for route

Lists all handlers defined for route. This will list all handlers defined for route, route's scope (if it has been defined on route) and route's context.
```
php nishchay route {path} -handler
```

##### Controller

Controller command is used find all controller defined in an application, annotation defined on controller class or method. It can also be used to create empty controller, CRUD controller and controllers based on template.

###### List controller

Running controller command without any parameter lists all controllers defined in application. This prints information as shown below

*   Controller full class path
*   Context it belongs to
*   Number of route exists in controller

```
php nishchay controller
```

###### Specific controller information

Passing controller full class path after controller command prints information of controller same as above command.

```
php nishchay controller {class}
```
###### Annotation defined on controller class

Passing `-annotation` following controller class will lists all annotation defined on controller class. This will print annotation name and parameter syntax.
```
php nishchay controller {class} -annotation
```
###### Annotation defined on controller method

We can also find list of annotations defined on controller method. To view this information just use below command:
```
php nishchay {class} -annotation -method {method}
```
###### Create empty controller

When we create empty controller using console command, Controller with specified class will be created at appropriate location based on full class name. Required annotation also be created on class declaration.

Created controller will comes with one method called `toDo` along with syntax for route annotation. This skelton method is shown below:

```php
/**
* Route(path='toDo')
*/
public function toDo()
{
   // REMOVE OR EDIT THIS ROUTE
}
```

Command to create empty controller is shown below:

```
php nishchay controller -create {class}
```

###### Create CRUD controller

To create controller which have create, update, delete and read operation. We just need to pass `-crud` parameter after class name.
```
php nishchay controller -create {class} -crud
```
When we execute above command, Nishchay will ask for route. Nishchay will not check if route already exists or not, it will directly creates controller with route prefix as entered by us. Below method with route will be created upon successful execution of command.

| Method | Route | HTTP |
| --- | --- | --- |
| index | [routePrefix]/ | GET |
| create | [routePrefix]/ | POST |
| fetch | [routePrefix]/{id} | GET |
| update | [routePrefix]/{id} | PUT |
| delete | [routePrefix]/{id} | DELETE |

###### Create controller from template

Apart from creating empty or crud controller, Nishchay also has predefined controllers which can be created using `-template` parameter.

Some templates have more than one controller, in that case Nishchay will ask us to create all controller or specific controller. If we choose specific then before creating each controller console will be prompted with answer yes or no for each controller creation. If there are multiple controllers in template then specify namespace within which all controllers need to be created or pass class name in case template having only one controller.

```
php nishchay controller -create {classOrNamesapce} -template {templateName}
```

We can create controllers from following list templates.

<table class="table table-bordered">
    <thead>
        <tr>
            <th>Template name</th>
            <th>Class list</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>employee</td>
            <td>
                <ul>
                    <li>Employee</li>
                    <li>Attendance</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>hostel</td>
            <td>
                <ul>
                    <li>Hostel</li>
                    <li>Fees</li>
                    <li>Building</li>
                    <li>Room</li>
                    <li>Guest</li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>

##### Entity

Using console command we can also find list of entities exists in an application. We can also find list of properties, derived properties, triggers of specific entity. Console command also allows creating empty, CRUD entity or entity from template.

###### List all entities

Just passing `entity` command without any parameter will list all entities exists in an application.
```
php nishchay entity
```

This will print full class name with table name it is mapped to in DB.

###### Entity properties

To find out list of properties and its data type just pass `{class} -property`.

```
php nishchay entity {class} -property
```

###### Derived properties
```
php nishchay entity {class} -derived
```

Upon executing above command, it will print derived properties of entity class. If there is no derived properties exist in entity class then will print error: `Derived property not found for: {class}`.

###### Identity property

Identity property is property which uniquely identifies each record in entity class, to know identity property of any entity class use below command.

```
php nishchay entity {class} -identity
```

###### Triggers

Just like other command to find properties, derived property and etc, trigger can be found by passing `-trigger` parameter.

```
php nishchay entity {class} -trigger
```

This will print details related to trigger like its callback method name, for which it is defined for(insert, update, remove), when it need to be executed and its priority.

###### Create empty entity

Creating empty entity means class with only one property which is identity property of class. This class also has `@entity` annotation defined on it and it will be mapped to table name as base class name.

Suppose we are creating an empty entity called `UserAddress` then its identity property will be created with `userAddressId`. Command to create empty entity is shown below.

```
php nishchay entity -create {class}
```

###### Create CRUD entity

CRUD entity means it will have following properties along with identity property.

<table class="table table-bordered">
    <thead>
        <tr>
            <th class="col-md-3">Property name</th>
            <th class="col-md-3">Data type</th>
            <th class="col-md-6">Use</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>createdAt</td>
            <td>datetime</td>
            <td>Date and time when record was inserted into table</td>
        </tr>
        <tr>
            <td>createdBy</td>
            <td>int</td>
            <td>Id of use who inserted record</td>
        </tr>
        <tr>
            <td>updatedAt</td>
            <td>datetime</td>
            <td>Date and time when record was ;ast updated. This should be changed on every update</td>
        </tr>
        <tr>
            <td>updatedBy</td>
            <td>int</td>
            <td>Id of user who last updated record</td>
        </tr>
        <tr>
            <td>deletedAt</td>
            <td>datetime</td>
            <td>Date and time when record was deleted. This should be changed when record is soft deleted</td>
        </tr>
        <tr>
            <td>deletedBy</td>
            <td>int</td>
            <td>Id of user who has deleted record</td>
        </tr>
        <tr>
            <td>isDeleted</td>
            <td>boolean</td>
            <td>Whether record is soft delete. Default to _false_</td>
        </tr>
    </tbody>
</table>

Command to create CRUD entity is shown below:

```
php nishchay entity -create {class} -crud
```

###### Create Entity from template

Entities can also be created using template. There can be multiple or single entity in template. If there are multiple entities in template, console asks us whether we want create all of entities or specific. Command to create entity from template is shown below.

```
php nishchay entity -create {classOrNamespace} -template {templateName}
```

If template has only one entity then pass only namespace or pass entity full class name. For template which creates multiple entities, those entities will be created in provided namespace.

<table class="table table-bordered">
    <thead>
        <tr>
            <th>Template name</th>
            <th>Class list</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>employee</td>
            <td>
                <ul>
                    <li>Employee</li>
                    <li>Attendance</li>
                    <li>Salary</li>
                    <li>Appraisal</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>hostel</td>
            <td>
                <ul>
                    <li>Hostel</li>
                    <li>Fees</li>
                    <li>Furniture</li>
                    <li>Building</li>
                    <li>Room</li>
                    <li>Guest</li>
                    <li>Mess</li>
                    <li>Visitor</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>category</td>
            <td>Category</td>
        </tr>
        <tr>
            <td>tree</td>
            <td>Tree</td>
        </tr>
        <tr>
            <td>user</td>
            <td>
                <ul>
                    <li>User</li>
                    <li>UserPassword</li>
                    <li>Session</li>
                    <li>UserPrivacy</li>
                    <li>PrivacyMember</li>
                    <li>UserPermission</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>activity</td>
            <td>
                <ul>
                    <li>Activity</li>
                    <li>AfftectedEntity</li>
                    <li>AfftectedProperty</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>message</td>
            <td>
                <ul>
                    <li>Thread</li>
                    <li>ThreadMember</li>
                    <li>Message</li>
                    <li>MessageAttachment</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>post</td>
            <td>
                <ul>
                    <li>Post</li>
                    <li>PostCategory</li>
                    <li>Attachment</li>
                    <li>Comment</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>asset</td>
            <td>
                <ul>
                        <li>Asset</li>
                        <li>LifeCycle</li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>

##### Events

Events are executed before & after controller & route. These events can be defined for global, scope and context. Using console command we can find events belongs to global, scope or context.

###### List all events

To view all events defined within application execute below command.
```
php nishchay event
```
###### Context events

Events belongs to specific context can be found by passing context parameter with context name.

```
php nishchay event -context {contextName}
```

###### Scope events

Events belongs to specific scope can be found by passing scope parameter with scope name.

```
php nishchay event -scope {scopeName}
```

###### Global events

Just passing `-global` following event command lists events defined for global.

```console
php nishchay event -global
```