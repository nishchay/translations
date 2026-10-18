#### Data - Entity Query

##### Introduction

Entity Query builds a query from your entity classes. You work with entity classes, aliases and property names, and Entity Query turns them into table and column names, runs the query and gives you entity objects back.

[Entity manager](/learningCenter/data/entity) can fetch every record or a single record by its identity property. Entity Query is for everything else: conditions, joins, ordering, grouping, limits, and updating or removing many records at once.

Entity Query is built on top of the [Query Builder](/learningCenter/data/queryBuilder). Conditions, operators, joins and ordering follow the same rules, so this page shows each option with entity examples and points to Query Builder only where nothing changes.

All examples on this page use these two entities.

```php
#[Entity(name: 'Employee')]
class Employee
{
    #[Identity]
    #[DataType(type: 'int', readOnly: true)]
    public $employeeId;

    #[DataType(type: 'string', length: 50)]
    public $firstName;

    #[DataType(type: 'int')]
    public $age;

    #[DataType(type: 'int')]
    public $salary;

    #[DataType(type: 'int')]
    public $departmentId;
}

#[Entity(name: 'Department')]
class Department
{
    #[Identity]
    #[DataType(type: 'int', readOnly: true)]
    public $departmentId;

    #[DataType(type: 'string', length: 50)]
    public $departmentName;
}
```

##### Create instance

`EntityQuery` is in the `Nishchay\Data` namespace. It accepts one optional argument, the database connection name. Without it, the default connection is used.

```php
$entityQuery = new Nishchay\Data\EntityQuery();
$mainQuery = new Nishchay\Data\EntityQuery('main');
```

Create a new instance for every query.

##### Set entity and alias

`setEntity` tells Entity Query which entity to read from. The second argument is the alias you will use in every property name afterwards.

```php
$entityQuery->setEntity(Employee::class, 'emp');
```

If you leave the alias out, the class name without its namespace is the alias, which is `Employee` here. A short alias is easier to read. `setTable` does the same as `setEntity`.

The class must be a registered entity, otherwise an exception is thrown.

##### Choose what to fetch

You have to say what should be fetched. Entity Query does not fetch anything by default, and `get` throws `Property to fetch is not set.` if you skip this step.

###### Whole entity

Pass only the alias to fetch every property of that entity.

```php
$records = (new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->setProperty('emp')
    ->get();
```

###### Specific properties

Prefix each property with its alias, and pass a list to fetch more than one.

```php
$records = (new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->setProperty(['emp.firstName', 'emp.age'])
    ->get();
```

Only these two properties are filled on the returned objects. A name without a dot is read as an alias, so `setProperty('firstName')` fails with `Mapping not found for class alias [firstName]`.

An unknown property name is not checked by Entity Query. It reaches the database, which reports `Unknown column`.

###### All except some properties

`setUnfetchable` takes the properties you do not want. Use it together with the alias to fetch everything else.

```php
$records = (new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->setProperty('emp')
    ->setUnfetchable('emp.salary')
    ->get();
```

Pass an array to leave out several properties.

```php
->setUnfetchable(['emp.salary', 'emp.age'])
```

###### Derived properties

A [derived property](/learningCenter/data/entity) needs extra joins, so it cannot be fetched through `setProperty`. Use `setDerivedProperty`. Without an alias, the property is read from the main entity.

```php
$entityQuery->setDerivedProperty('department');
```

Call it once for each derived property. Two cases are not supported by Entity Query and throw an exception: a derived property that returns a list of values, and a derived property that returns a whole entity. Fetch those through [Entity manager](/learningCenter/data/entity).

###### Lazy properties

Call `setLazy(true)` to have lazy properties loaded too. It works only when the fetched properties belong to one entity.

```php
$entityQuery->setLazy(true);
```

##### Fetch records

###### All matched records

`get` runs the query and returns the records as an iterator of entity objects.

```php
foreach ($records as $employee) {
    echo $employee->firstName;
}
```

###### First record

`getOne` returns the first matched record, or `false` when nothing matched. It sets `LIMIT 1` for you.

```php
$employee = (new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->setProperty('emp')
    ->setCondition('emp.employeeId', 2)
    ->getOne();

if ($employee === false) {
    # not found
}
```

##### Where

`setCondition` is the same method as the Query Builder one, so every [where option](/learningCenter/data/queryBuilder) works here. The difference is the left operand. The table is aliased, so write the property as `alias.property`.

**NOTE** `setCondition` does not check that the alias or the property exists. A wrong name fails in the database.

###### Simple

```php
$entityQuery->setCondition('emp.firstName', 'Asha');
```

```sql
`emp`.`firstName` = :i0b1
```

###### Multiple conditions

```php
$entityQuery->setCondition([
    'emp.departmentId' => 2,
    'emp.age' => 45
]);
```

```sql
`emp`.`departmentId` = :i0b1 AND `emp`.`age` = :i0b2
```

###### Operators

Put the operator at the end of the property name.

```php
$entityQuery->setCondition('emp.age>', 25);
```

```sql
`emp`.`age` > :i0b1
```

###### IN, NOT IN and BETWEEN

```php
$entityQuery->setCondition([
    'emp.departmentId' . Query::IN => [1, 2],
    'emp.age' . Query::BETWEEN => [25, 45]
]);
```

```sql
`emp`.`departmentId` IN (:i0b1,:i0b2) AND `emp`.`age` BETWEEN :i0b3 AND :i0b4
```

Use `Query::NOT_IN` for NOT IN.

###### OR

Multiple conditions are joined with AND. Add `'OR'` as the last element to join them with OR.

```php
$entityQuery->setCondition([
    'emp.age<' => 25,
    'emp.salary>' => 80,
    'OR'
]);
```

```sql
`emp`.`age` < :i0b1 OR `emp`.`salary` > :i0b2
```

###### Groups of conditions

An array without a key makes a bracketed group, exactly as in Query Builder.

```php
$entityQuery->setCondition([
    [
        'emp.departmentId' => 1,
        'emp.age>' => 40
    ],
    'emp.salary>' => 80,
    'OR'
]);
```

```sql
(`emp`.`departmentId` = :i0b1 AND `emp`.`age` > :i0b2) OR `emp`.`salary` > :i0b3
```

###### Compare two columns

Add `Query::AS_COLUMN` to the end of the left operand and the value is used as a column.

```php
$entityQuery->setCondition('emp.age' . Query::AS_COLUMN, 'emp.salary');
```

```sql
`emp`.`age` = `emp`.`salary`
```

###### Use a value or a function as it is

`Query::AS_IT_IS` keeps the right operand as written and `Query::LEFT_AS_IT_IS` does the same for the left operand. Build the pieces with `quote` and `bindValue` so that names are escaped and values are bound. Both are available on `EntityQuery`.

```php
$entityQuery->setCondition(
    'emp.salary' . Query::AS_IT_IS,
    $entityQuery->quote('emp.age') . ' + ' . $entityQuery->bindValue(20)
);
```

```sql
`emp`.`salary` = `emp`.`age` + :i0b1
```

Read more about [operator order and the other markers](/learningCenter/data/queryBuilder).

##### Join

`addJoin` is the Query Builder join with two differences: the key is a **class name** instead of a table name, and the **alias is required**. Write the alias in brackets after the class.

```php
$entityQuery
    ->setEntity(Employee::class, 'emp')
    ->addJoin([
        Department::class . '(dep)' => 'departmentId'
    ]);
```

```sql
LEFT JOIN `Department` AS `dep` ON `dep`.`departmentId` = `emp`.`departmentId`
```

Without the alias, Entity Query throws `Invalid join rule [...]. Missing alias.`.

###### Same column name in both entities

A string value means both entities are joined on the column of that name, as in the example above.

###### Different column names

Use an array. The key is the column of the joined entity, and the value is the column of the main entity.

```php
->addJoin([
    Department::class . '(dep)' => ['departmentId' => 'emp.departmentId']
])
```

Add the alias to a column when it belongs to another entity, as `emp.departmentId` does above. Use `Query::AS_VALUE` at the end of the left operand to compare with a value instead of a column.

```php
->addJoin([
    Department::class . '(dep)' => ['departmentId', 'departmentName' . Query::AS_VALUE => 'Tech']
])
```

###### More than one join

```php
->addJoin([
    Department::class . '(dep)' => 'departmentId',
    Zone::class . '(zone)' => ['zoneId' => 'dep.zoneId']
])
```

###### Join type

Joins are LEFT joins by default. Put a join type constant in front of the class name to change it.

```php
->addJoin([
    Query::INNER_JOIN . Department::class . '(dep)' => 'departmentId'
])
```

| Constant | Join |
|---|---|
| `Query::INNER_JOIN` | INNER |
| `Query::LEFT_JOIN` | LEFT (default) |
| `Query::RIGHT_JOIN` | RIGHT |
| `Query::CROSS_JOIN` | CROSS |
| `Query::FULL_JOIN` | FULL |

###### Join only to filter

A joined entity does not have to be fetched. Use it in a condition and fetch only the main entity.

```php
$records = (new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->addJoin([Department::class . '(dep)' => 'departmentId'])
    ->setProperty('emp.firstName')
    ->setCondition('dep.departmentName', 'Tech')
    ->get();
```

##### Records with joins

When the query fetches properties of **one** entity, each record is that entity object. With **two or more** entities, each record holds one object per alias.

```php
$records = (new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->addJoin([Department::class . '(dep)' => 'departmentId'])
    ->setProperty(['emp', 'dep'])
    ->get();

foreach ($records as $row) {
    echo $row->emp->firstName . ' (' . $row->dep->departmentName . ')';
}
```

You can also pick single properties from each entity.

```php
->setProperty(['emp.firstName', 'dep.departmentName'])
```

If a LEFT join finds no match, the joined alias is `null`, so check it before use.

```php
echo $row->dep === null ? 'No department' : $row->dep->departmentName;
```

An entity you join but never put in `setProperty` is not part of the result.

##### Group by

Pass one property, or a list.

```php
$entityQuery->setGroupBy('emp.departmentId');
$entityQuery->setGroupBy(['emp.departmentId', 'emp.age']);
```

Each call adds to the previous ones.

##### Order by

Pass one property to sort ascending, or a list. Use the property as key and `ASC` or `DESC` as value to choose the direction.

```php
$entityQuery->setOrderBy('emp.age');
$entityQuery->setOrderBy([
    'emp.departmentId' => 'ASC',
    'emp.age' => 'DESC'
]);
```

##### Limit

The first argument is how many records, and the second is the offset.

```php
$entityQuery->setLimit(10);      # first 10
$entityQuery->setLimit(10, 20);  # 10 records after skipping 20
```

##### Having, counts and other query builder features

`EntityQuery` has no `setHaving`, `count`, `sum`, `max`, `min`, `addUnionQuery` or `getSQL` of its own. `getQueryBuilder` returns a copy of the underlying `Query` with everything you have set so far, and you can use these on it.

```php
$query = $entityQuery
    ->setEntity(Employee::class, 'emp')
    ->setCondition('emp.age>', 20)
    ->getQueryBuilder();

$total = $query->count();
$salary = $query->sum('emp.salary');
echo $query->getSQL();
```

Use it for `setHaving` too, see [Having](/learningCenter/data/queryBuilder). Records fetched this way come from the Query Builder, so they are plain objects and not entity objects.

The query you get is a copy. Changes made on it do not reach the `EntityQuery`.

##### Cache

`setCache` caches the result. Pass `true` to build the cache key from the query, or pass your own key. The second argument is the lifetime in seconds, where `0` means no expiry.

```php
$entityQuery->setCache(true, 3600);
$entityQuery->setCache('top-employees', 600);
```

##### Update records

`EntityManager` updates one record. Entity Query updates **every record that matches your conditions** in one statement, and returns how many were updated.

Set the new values with `setPropertyWithValue`. Name the property **without** an alias, because only properties of the main entity can be updated.

```php
$updated = (new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->setPropertyWithValue([
        'salary' => 55,
        'departmentId' => 3
    ])
    ->setCondition('emp.age>', 40)
    ->update();
```

Before the query runs, Entity Query does the same work as Entity manager:

* The new values are validated against the entity rules.
* Callbacks that run before and after a change are executed. If a before callback returns `false`, nothing is updated and `update` returns `false`.

An exception is thrown when you try to update a property that does not exist in the main entity, a derived or static property, or a read only property (an identity property is usually read only).

Without a condition, **every** record is updated.

###### Update using a join

A join can be used to decide which records to update. Only the main entity is changed.

```php
(new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->addJoin([Department::class . '(dep)' => 'departmentId'])
    ->setPropertyWithValue(['salary' => 55])
    ->setCondition('dep.departmentName', 'Sales')
    ->update();
```

##### Remove records

Set the main entity and the conditions, then call `remove`. It returns the number of records removed.

```php
(new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->setCondition('emp.employeeId', 5)
    ->remove();
```

###### Remove using a join

```php
(new EntityQuery())
    ->setEntity(Employee::class, 'emp')
    ->addJoin([Department::class . '(dep)' => 'departmentId'])
    ->setCondition('dep.departmentName', 'Closed')
    ->remove();
```

This removes employees only, not departments.

**NOTE** `remove` without a condition removes every record of the entity.

##### Quick reference

| Method | Purpose |
|---|---|
| `setEntity` / `setTable` | Main entity and its alias |
| `setProperty` | Properties or whole entities to fetch |
| `setUnfetchable` | Properties to leave out |
| `setDerivedProperty` | Derived property to fetch |
| `setLazy` | Fetch lazy properties |
| `setCondition` | Where |
| `addJoin` | Join entities (alias required) |
| `setGroupBy`, `setOrderBy`, `setLimit` | Grouping, sorting, limiting |
| `setCache` | Cache the result |
| `get`, `getOne` | Fetch records |
| `setPropertyWithValue`, `update` | Update records |
| `remove` | Remove records |
| `quote`, `bindValue` | Build expressions safely |
| `getQueryBuilder` | Copy of the underlying `Query` (having, count, SQL) |

##### Common errors

| Message | Cause |
|---|---|
| `Property to fetch is not set.` | `get` was called before `setProperty` |
| `Mapping not found for class alias [x]` | `setProperty` got a name without a dot that is not an alias |
| `Invalid join rule [...]. Missing alias.` | A join class has no `(alias)` |
| `Class [...] is not registered entity.` | The class is not an entity |
| `Can not update read only property` | `setPropertyWithValue` got a read only property |
| `Property to update [...] does not exist.` | `setPropertyWithValue` got an unknown property |
