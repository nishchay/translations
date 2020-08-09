#### Data - Entity Query

Entity query allow us to create query based on Entity class. Entity Query then converts entity based query to actual query. As Entity Manager allows only fetching all records or single by its identity property, Entity query allows to fetch records by applying conditions, joins, orders and etc.

###### Create Instance

Entity query accepts only one optional argument which is database connection name, just as in Query builder if we omit connection name it will use default database connection.
```php
$entityQuery = new Nishchay\Data\EntityQuery();
```
###### Set entity

Using `setEntity` method we can set entity.
```php
$entityQuery->setEntity(User::class)
```
To set an alias for entity, we can pass it in second argument of `setEntity` method. By default `EntityQuery` uses base name of entity class.

```php
# Alias will be: User (base class name)
$entityQuery->setEntity(User::class);

# Alias will be: u
$entityQuery->setEntity(User::class, 'u');
```

###### Fetch whole entity
To fetch all properties of entity, pass only alias name of entity class.
```php
$entityQuery = new EntityQuery();
$entityQuery->setEntity(User::class, 'user')
    ->setProperty('user');
```

Based on alias name, EntityQuery find all properites of entity class of alias and sets it to list of properties which needs to be fetched.

###### Set property to be fetched
Its mandatory to set which property should be fetched for Entity Query. Entity Query does not fetches all property of Entity class by default.
```php
$entityQuery->setProperty(['user.userId', 'user.firstName', 'user.lastName'])
```
If want to set only one property then pass only string otherwise pass list of properties in an array. 
Properites being set should be prefixed with entity alias name. This is because if we do not specify it with alias name then it will be counted as entity alias name and EntityQuery tries to find entity class for alias name to fetch all properties of it.

###### Set derilved property to be fetched
Using `setDerivedProperty` we can set which derived property which needs to fetched. As derived property requires additional joins based on relative property, derived can not be set usig `setProperty` method.

**NOTE** `EntityQuery` does not allow fetching whole derived entity, because it requires separate query to set those property.

```php
$entity->setDerivedProperty('department');
```

If want to set multiple properties, call `setDerivedProperty` multiple times.

###### Set property which should not be fetched
We can set property which should not be fetched. This is required when we want to fetch whole entity except one or more property. Same as setting property to fetch, this also requires alias name along with property name.
```php
$entityQuery = new EntityQuery();
$entityQuery->setEntity(User::class, 'user')
    ->setUnfetchable('user.birthDate');
```
For above all property will be fetched except _birthDate_. Pass list of properties in an array, if we want to set multiple properties which needs not be fetched.

###### Apply Condition to fetch record

Applying condition is same as Query Builder `setCondition`, learn more about [here](/learningCenter/data/queryBuilder?topic=setCondition).

**NOTE** Unlike all of above methods, `setCondition` does not validates property is valid or not. It also does not check if passed property is entity alias or property itself.

###### Join

Adding join is same is Query Builder join except it requires class name instead of table name and alias is required. Omitting alias for join table will result in an error. Lear more about join [here.](/learningCenter/data/queryBuilder?topic=addJoin)

See below example for how join can be added:

```php
$entity->addJoin([
    Department::class . '(department)' => 'departmentId',
    Zone::class . '(zone)' => ['zoneId' => 'department.zoneId']
]);
```


###### Order By, Group By & Having

All these are same as query builder, please learn [here.](/learningCenter/data/queryBuilder)

##### Update entity record
Using `EntityQuery` we can update multiple record at same while `EntityManager` allows updating only one record at a time.

**NOTE** Using `EntityQuery` we can update properties of main entity only which was set using `setEntity` method. So while we add properties with value which need to updated, property should be passed without its alias name.

Using `setPropertyWithValue` we can set list of properties which needs to updated. Validation of property value is validated when we call `update` method.

See below code for better understanding

```php
$entity->setPropertyWithValue([
    'firstName' => 'New first Name',
    'departmentId' => 3 # New department Id
])
->update();
```
Once query has been executed `update` method returns number of records updated.

##### Remove entity record
To remove record of main entity either by join or without join is same as `Query` class. We only need to set main entity, conditions and then call `remove` method.

```php
$entity->setEntity(User::class)
    ->setCondition('departmentId', 23)
    ->remove();
```

Above code will remove all users who are belongs to `departmentId  = 23`;

##### How multiple entities returned by EntityQuery[time:120]
If only entity is fetched then entities rows are directly accessible but if there's more than one returned by entity query, it will be assigned entity class alias. See below example

```php
$entity->setEntity(User:class)
    ->addJoin([
        Department::class(department) => 'userId'
    ])
    ->setProperty(['user', 'department'])
    ->get();
```

This will fetch `User` and `Department` whole entity and are returned like this:

```php
[
    [
        'user' => UserEnittyObject,
        'department' => DepartmentEntityObject
    ],
    [
        'user' => UserEnittyObject,
        'department' => DepartmentEntityObject
    ]
    . . .
]
```

When we iterate over Entity records, each row can accessed by `$row->user`. See below example how we can print all users and its department name from above records.

```php
foreach($records as $row) {
    echo $row->user->fullName. '(' . $row->department->departmentName . ')';
}
```