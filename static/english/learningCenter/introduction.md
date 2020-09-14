#### Nishchay means 'decision'

`Nishchay` is an open structure PHP framework released under New BSD license in 2020 developed by `Bhavik Patel`.
Name of the framework chosen from 'To do anything, we only need to decide and start working on it'.
As the name suggests Nishchay has number of features which helps you to achieve your aim.

##### Open structure
1. Open structure allows you to create any kind application structure like, MVC, HMVC or anything.
2. We can also create our application structure.
3. This makes framework most compatible than other framework.
4. In other framework whenever application structure changes it brings incompatibility with its previous version, this does not happens with nishchay framework.


##### Maintenance modes

Put application into maintenance based on following:

1. Browser based maintenance, also allows maintenance for version grater than, less than or version range.
2. Time range for daily maintenance.
3. Date range for future maintenance plan.
4. Based on callback method

Allow routes even in maintenance mode based on following:

1. List of routes.
2. Routes which matches regex pattern.
2. Routes belongs to scope.


##### Session

Nishchay has various kind of sessions are which are listed below:

1. Read limit which expires after its read limit is reached.
2. Till next request which expires after given number of requests
3. Scope session which is available for given scope only.
4. Conditional which works based on condition.

Session data can be stored to:

1. Files
2. Database
3. Cache

##### Web service

Make whole application web service or selected routes. Apart from REST full routes which also supports JSON & XML response types, we service adds more features to routes.

1. Web service response can be filtered by user who uses web services. This is done with the help fields demand.
2. Enable or disable services to be accessed by token. This token can be configured to be sent in Header or request parameter.
3. Create your own response format.
4. Web services also allows setting always returning and supported response fields.

##### Database

1. Entity class which maps database table allows defining readonly, required, derived (Value derives from another entity) and more types of property.
2. Relate entity to another entity, add extra property to row.
3. Event which executes before or after insert update or delete.
3. Entity query builder allows creating query based on entity class, using this we can create simple to complex query.
4. Just set `encrypt = true` and value of property is stored as encrypted in DB and decrypted back when fetching.
5. In built caching for Query builder and Entity query builder which if configured then it fetches data from cache first and then database if not found in cache.
6. Database manager which allows manipulating database like creating or altering database table.

##### Console command

1. Create controller or entity using console command. We can create empty, crud or from templates.
2. Using console command we can find list of routes, controller, entities and events within application. 
3. Use console command find information regarding route, controller, entities and more.
3. Console command can also be used for running routes.

**THERE'S SO MANY OTHER THINGS, GO THOROUGH EACH TOPIC FROM DOCUMENTATION NAVIGATION MENU**