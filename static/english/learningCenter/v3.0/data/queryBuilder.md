#### Data - Query Builder

##### Introduction

Query builder creates and runs SQL statements **directly on database tables**. It does not use entity classes, so it works for any table, including tables that have no entity. The same code runs on every supported database, because column escaping and statement syntax are handled for each connection.

Values are always sent to the database as bound parameters, never pasted into the statement. This protects the query from SQL injection.

To work with entity classes instead of tables, see [Entity Query](/learningCenter/data/entityQuery).

All examples use these two tables.

```sql
user (userId, firstName, age, departmentId, city)
department (departmentId, departmentName, zoneId)
```

##### Creating instance

`Query` is in the `Nishchay\Data` namespace. It accepts one optional argument, the name of a database connection. Without it, the default connection is used. An instance works only with the connection it was created for.

```php
use Nishchay\Data\Query;

$query = new Query();        # default connection
$query = new Query('main');  # connection named main
```

Every call that runs a statement (`get`, `getOne`, `insert`, `update`, `remove`, `count` and the aggregate methods) **clears the instance**, including the table. Set the table again before reusing it, or create a new instance.

##### Set table

`setTable` sets the table to work on. The second argument is an optional alias for the table.

```php
$query->setTable('user');
$query->setTable('user', 'u');
```

When you use an alias, write columns with it.

```php
$query->setTable('user', 'u')
    ->setColumn('u.firstName')
    ->setCondition('u.age>', 40)
    ->get();
```

```sql
SELECT `u`.`firstName` FROM `user` AS `u` WHERE `u`.`age` > :i0b1
```

The table can also be a sub query, see _Sub query_ below.

##### Select

###### All columns

```php
$records = $query->setTable('user')->get();
```

`get` runs the statement and returns **an array of objects**, one per row. It returns an empty array when nothing matched.

```php
foreach ($records as $row) {
    echo $row->firstName;
}
```

Pass a PDO fetch mode to `get` to receive another format, for example `get(PDO::FETCH_ASSOC)` for arrays.

###### Specific columns

```php
$query->setTable('user')
    ->setColumn(['userId', 'firstName', 'lastName'])
    ->get();
```

Pass a string for a single column. Every call adds to the columns set before it, so the next code selects the same columns.

```php
$query->setTable('user')
    ->setColumn('userId')
    ->setColumn('firstName')
    ->setColumn('lastName')
    ->get();
```

###### First row

`getOne` returns the first matched row as an object, or `false` when there is none. It applies `LIMIT 1`.

```php
$user = $query->setTable('user')
    ->setColumn(['userId', 'firstName'])
    ->setCondition('userId', 1)
    ->getOne();
```

###### Column alias

Use a string key to name the column in the result.

```php
$query->setTable('user')
    ->setColumn(['userId', 'fname' => 'firstName'])
    ->get();
```

Each row has `userId` and `fname`.

###### Function or expression as column

Column names are escaped by default, so a function would be quoted as a column. Add `Query::AS_IT_IS` at the end of the alias to keep the value exactly as written. An alias is required.

```php
$query->setTable('user')
    ->setColumn([
        'total' . Query::AS_IT_IS => 'SUM(' . $query->quote('age') . ' + ' . $query->bindValue(2) . ')'
    ])
    ->get();
```

```sql
SELECT SUM(`age` + :i0b1) AS `total` FROM `user`
```

`quote` escapes a column or table name for the current connection. `bindValue` stores a value and returns its placeholder. Always build expressions with these two instead of joining raw strings and values.

###### Count, sum, max, min and average

These methods run the statement and return the result directly. They respect the table, joins and conditions you have set.

```php
$total = $query->setTable('user')->setCondition('age>', 25)->count();
$all = $query->setTable('user')->count('city');   # counts rows where city is not NULL

$sum = $query->setTable('user')->sum('age');
$max = $query->setTable('user')->max('age');
$min = $query->setTable('user')->min('age');
$avg = $query->setTable('user')->average('age');
```

`count` returns an integer. `sum` and `average` return the value the database gives, usually a string. `max`, `min` and `average` need rows to work on, so make sure the conditions can match something.

##### Where

`setCondition` adds a WHERE clause. The first argument is a column name, or an array for many conditions, and the second is the value. Calling it again adds one more group joined with AND.

```php
$query->setCondition('firstName', 'Asha');
$query->setCondition(['firstName' => 'Asha', 'age' => 30]);
```

```sql
`firstName` = :i0b1
`firstName` = :i0b2 AND `age` = :i0b3
```

The array keys must be column names (or expressions, see below). The value is always bound, so it is safe to use input from the user.

###### Operators

Put the operator at the end of the column name. Without an operator, `=` is used.

```php
$query->setCondition('age>', 18);
$query->setCondition(['age>=' => 18, 'age<>' => 30, 'firstName like' => 'A%']);
```

```sql
`age` > :i0b1
`age` >= :i0b2 AND `age` <> :i0b3 AND `firstName` like :i0b4
```

Supported operators are `=`, `>`, `<`, `>=`, `<=`, `<>`, `!=` and `like`.

###### NULL

A `null` value becomes `IS NULL`. With `<>`, `!=` or `!` it becomes `IS NOT NULL`.

```php
$query->setCondition('city', null);
$query->setCondition('city<>', null);
```

```sql
`city` IS NULL
`city` IS NOT NULL
```

###### IN and NOT IN

Add `Query::IN` or `Query::NOT_IN` at the end of the column and pass an array.

```php
$query->setCondition([
    'age' . Query::IN => [21, 23, 25],
    'userId' . Query::NOT_IN => [1, 2]
]);
```

```sql
`age` IN (:i0b1,:i0b2,:i0b3) AND `userId` NOT IN (:i0b4,:i0b5)
```

An empty array throws `Empty value for the [column]`, so check the list before you build the clause.

###### BETWEEN

Pass an array of exactly two values.

```php
$query->setCondition(['age' . Query::BETWEEN => [18, 60]]);
```

```sql
`age` BETWEEN :i0b1 AND :i0b2
```

###### Compare with another column

Add `Query::AS_COLUMN` and the value is treated as a column.

```php
$query->setCondition('userId' . Query::AS_COLUMN, 'age');
```

```sql
`userId` = `age`
```

###### Use the value as it is

Add `Query::AS_IT_IS` and the value goes into the statement unchanged and unescaped. Build it with `quote` and `bindValue`.

```php
$query->setCondition('age' . Query::AS_IT_IS, 'SUM(' . $query->quote('userId') . ' + 2)');
```

```sql
`age` = SUM(`userId` + 2)
```

###### Use the left side as it is

The left side is read as a column name. To use a function or an expression there, add `Query::LEFT_AS_IT_IS` at the end. Write the operator yourself, before the marker.

```php
$query->setCondition('SUM(' . $query->quote('age') . ' + 2)=' . Query::LEFT_AS_IT_IS, 200);
```

```sql
SUM(`age` + 2)=:i0b1
```

###### Marker order

When an operator and a marker are used together, the operator comes first.

1. `Query::AS_COLUMN`
2. `Query::AS_IT_IS`
3. `Query::LEFT_AS_IT_IS`

```php
$query->setCondition('a>' . Query::AS_COLUMN, 'b');
```

```sql
`a` > `b`
```

###### OR

Conditions in one array are joined with AND. Put `'OR'` as the last element, without a key, to join them with OR.

```php
$query->setCondition([
    'firstName' => 'Asha',
    'age' => 28,
    'OR'
]);
```

```sql
`firstName` = :i0b1 OR `age` = :i0b2
```

`'AND'` can be written as the last element to make the intent clear.

**NOTE** The last element is used as it is, without a check. `'JOINER'` would produce `... JOINER ...` and fail in the database. Take care with the spelling.

###### Groups of conditions

An array without a key becomes a group in brackets.

```php
$query->setCondition([
    ['a' => 1, 'b' => 2],
    'c' => 3,
    ['d' => 1, 'e' => 2, 'f' => 3, 'OR'],
    'OR'
]);
```

```sql
(`a` = :i0b1 AND `b` = :i0b2) OR `c` = :i0b3 OR (`d` = :i0b4 OR `e` = :i0b5 OR `f` = :i0b6)
```

The first group joins with AND because it has no joiner at its end. The third group ends with `'OR'`, so its parts are joined with OR. The outer `'OR'` joins the groups.

###### Several calls

```php
$query->setCondition('a', 1)->setCondition('b', 2);
```

```sql
(`a` = :i0b1) AND (`b` = :i0b2)
```

##### Join

Set the table first, because the join uses it. `addJoin` takes an array. Each key is the table to join, and the value describes the `ON` clause.

###### Same column in both tables

A string means both tables have a column of that name.

```php
$query->setTable('user')
    ->addJoin(['department' => 'departmentId']);
```

```sql
SELECT * FROM `user`
LEFT JOIN `department` ON `department`.`departmentId` = `user`.`departmentId`
```

###### Different column names

Use an array. The key is the column of the joined table and the value is the column of the main table.

```php
$query->setTable('user')
    ->addJoin(['department' => ['id' => 'departmentId']]);
```

```sql
LEFT JOIN `department` ON `department`.`id` = `user`.`departmentId`
```

A column without a table name gets the joined table on the left and the main table on the right.

###### Join conditions

Join conditions follow the rules of `setCondition`, with one difference: in a join the **right side is a column** by default. Add `Query::AS_VALUE` at the end of the left side to compare with a value, which is bound.

```php
$query->setTable('user')
    ->addJoin([
        'department' => [
            'departmentId',                              # same column in both tables
            'zoneId>' . Query::AS_VALUE => 5,            # compared with the value 5
            'departmentName' . Query::IN => ['Sales', 'Tech']
        ]
    ]);
```

```sql
ON `department`.`departmentId` = `user`.`departmentId`
AND `department`.`zoneId` > :i0b1
AND `department`.`departmentName` IN (:i0b2,:i0b3)
```

Without `Query::AS_VALUE`, `'zoneId>' => 5` would compare `zoneId` with a **column** named `5`.

Do not add `Query::AS_COLUMN` in a join, it is the default. Likewise, do not add `Query::AS_VALUE` in `setCondition`.

Several conditions are joined with AND. To join them with OR, put the joiner at the end and prefix it with `Query::ON_JOINER`. The prefix is needed because in a join a plain string means a column name.

```php
->addJoin([
    'department' => [
        'departmentId',
        'zoneId' => 'zoneId',
        Query::ON_JOINER . 'OR'
    ]
])
```

```sql
ON `department`.`departmentId` = `user`.`departmentId` OR `department`.`zoneId` = `user`.`zoneId`
```

###### More than one table

```php
$query->setTable('user')
    ->addJoin([
        'department' => 'departmentId',
        'city' => 'cityId'
    ]);
```

```sql
LEFT JOIN `department` ON `department`.`departmentId` = `user`.`departmentId`
LEFT JOIN `city` ON `city`.`cityId` = `user`.`cityId`
```

###### Column of another table

When a column has a table name, the builder does not add one. This also works for the right side, so a join can use a table joined earlier.

```php
->addJoin([
    'department' => 'departmentId',
    'zone' => ['zoneId', 'code' => 'department.zoneCode']
])
```

```sql
LEFT JOIN `zone` ON `zone`.`zoneId` = `user`.`zoneId` AND `zone`.`code` = `department`.`zoneCode`
```

###### Join type

Joins are LEFT joins unless you put a type in front of the table name.

```php
$query->setTable('user')
    ->addJoin([
        Query::INNER_JOIN . 'department' => 'departmentId'
    ]);
```

| Constant | Join |
|---|---|
| `Query::INNER_JOIN` | INNER |
| `Query::LEFT_JOIN` | LEFT (default) |
| `Query::RIGHT_JOIN` | RIGHT |
| `Query::CROSS_JOIN` | CROSS |
| `Query::FULL_JOIN` | FULL |

Your database must support the join you ask for. MySQL, for example, has no FULL join.

###### Alias of the joined table

Write the alias in brackets after the table name, and use it in conditions and columns.

```php
$query->setTable('user')
    ->setColumn(['user.firstName', 'd.departmentName'])
    ->addJoin(['department(d)' => 'departmentId']);
```

```sql
SELECT `user`.`firstName`,`d`.`departmentName`
FROM `user`
LEFT JOIN `department` AS `d` ON `d`.`departmentId` = `user`.`departmentId`
```

##### Group by

Pass one column or an array. Every call adds more columns.

```php
$query->setGroupBy('departmentId');
$query->setGroupBy(['departmentId', 'city']);
```

##### Having

`setHaving` works like `setCondition`, so every option in the _Where_ section can be used. The usual use is a condition on an aggregate. Use `Query::LEFT_AS_IT_IS` for the aggregate, or give it an alias with `Query::AS_IT_IS` and compare the alias.

```php
$query->setTable('user')
    ->setColumn(['departmentId', 'total' . Query::AS_IT_IS => 'COUNT(*)'])
    ->setGroupBy('departmentId')
    ->setHaving('COUNT(*)>' . Query::LEFT_AS_IT_IS, 1)
    ->get();
```

```sql
SELECT `departmentId`,COUNT(*) AS `total` FROM `user` GROUP BY `departmentId` HAVING COUNT(*)>:i0b1
```

##### Order by

Pass a column to sort ascending. Pass an array to sort by many columns, and use the column as key and `ASC` or `DESC` as value to choose the direction. Every call adds more columns.

```php
$query->setOrderBy('age');
$query->setOrderBy(['departmentId' => 'DESC', 'age']);
```

```sql
ORDER BY `age` ASC ,`departmentId` DESC ,`age` ASC
```

Start the value with `Query::AS_IT_IS` to use it without escaping, for example `setOrderBy([Query::AS_IT_IS . 'RAND()'])`.

##### Limit

The first argument is how many rows, and the second is how many rows to skip.

```php
$query->setLimit(5);       # first 5 rows
$query->setLimit(5, 10);   # 5 rows after skipping 10
```

##### Union

`addUnionQuery` adds another query to the result. Pass `true` as the second argument for `UNION ALL`, which keeps duplicates.

```php
$query->setTable('user')->setColumn('firstName')->setCondition('age>', 40);

$query->addUnionQuery(
    (new Query())->setTable('department')->setColumn('departmentName'),
    true
);

$records = $query->get();
```

```sql
SELECT `firstName` FROM `user` WHERE `age` > :i0b1
UNION ALL
SELECT `departmentName` FROM `department`
```

Both queries must select the same number of columns. The names in the result come from the first query.

##### Insert

Set the table and the values, then call `insert`. `insert` returns the id the database generated for the new row.

```php
$id = $query->setTable('user')
    ->setColumnWithValue([
        'firstName' => 'Asha',
        'age' => 28
    ])
    ->insert();
```

A single value can be set as `setColumnWithValue('firstName', 'Asha')`. Calling `insert` without any value throws `Column with its value are not set.`

`DateTime` values are converted to the database format for you.

###### Insert many rows

`setColumnWithMultipleValue` takes the column names, then a list of rows. Every row must have the same number of values as there are columns, otherwise an exception is thrown.

```php
$query->setTable('user')
    ->setColumnWithMultipleValue(['firstName', 'age'], [
        ['Asha', 28],
        ['Ravi', 45],
        ['Meena', 22]
    ])
    ->insert();
```

All rows are inserted by one statement. The returned id is the one of the first row.

##### Update

Set the table, the new values and the conditions, then call `update`. It returns how many rows were changed.

```php
$updated = $query->setTable('user')
    ->setColumnWithValue(['age' => 29, 'city' => 'Pune'])
    ->setCondition('userId', 1)
    ->update();
```

Markers at the end of the column name change how the value is used.

| Marker | Meaning |
|---|---|
| none | The value is bound |
| `Query::AS_COLUMN` | The value is a column name |
| `Query::AS_IT_IS` | The value is used as written |

```php
$query->setTable('user')
    ->setColumnWithValue([
        'age' . Query::AS_IT_IS => $query->quote('age') . ' + 1',   # age = age + 1
        'city' . Query::AS_COLUMN => 'firstName'                   # city = firstName
    ])
    ->setCondition('userId', 1)
    ->update();
```

Use `null` to store NULL, as in `setColumnWithValue('city', null)`.

**NOTE** An update with **no condition** is refused with `Condition is required to update record.` It prevents changing every row by mistake. To allow it, set `updateNoCondition` to `true` in `settings/configuration/database.php`.

###### Update using a join

Add a join and name the column of the joined table in `setColumnWithValue`.

```php
$query->setTable('user')
    ->addJoin(['department' => 'departmentId'])
    ->setColumnWithValue('department.zoneId', 77)
    ->setCondition('user.firstName', 'Asha')
    ->update();
```

This changes the joined `department` table. Use the table name in front of every column to make it clear which table is changed.

##### Remove

Set the table and the conditions, then call `remove`. It returns how many rows were removed.

```php
$removed = $query->setTable('user')
    ->setCondition('userId', 1)
    ->remove();
```

**NOTE** A remove with **no condition** is refused with `Condition is required to delete record.` Set `deleteNoCondition` to `true` in `settings/configuration/database.php` to allow it.

###### Remove using a join

A join can decide which rows are removed.

```php
$query->setTable('user')
    ->addJoin(['department' => 'departmentId'])
    ->setCondition('department.departmentName', 'Closed')
    ->remove();
```

##### Sub query

A `Query` instance can be used inside another query. Build it and pass it where the value would go. The values of the sub query are bound together with the main query.

###### In column

A sub query can be a column. The key is the alias and it is required.

```php
$counter = (new Query())
    ->setTable('user')
    ->setColumn(['total' . Query::AS_IT_IS => 'COUNT(*)'])
    ->setCondition('user.departmentId' . Query::AS_COLUMN, 'department.departmentId');

$records = $query->setTable('department')
    ->setColumn(['departmentName', 'members' => $counter])
    ->get();
```

```sql
SELECT `departmentName`,
    (SELECT COUNT(*) AS `total` FROM `user` WHERE `user`.`departmentId` = `department`.`departmentId`) AS `members`
FROM `department`
```

###### In where

A sub query can be the right side of a condition. It is `=` by default, and works with `Query::IN`, `Query::NOT_IN` and the comparison operators.

```php
$tech = (new Query())
    ->setTable('department')
    ->setColumn('departmentId')
    ->setCondition('departmentName', 'Tech');

$records = $query->setTable('user')
    ->setCondition(['departmentId' . Query::IN => $tech])
    ->get();
```

```sql
SELECT * FROM `user`
WHERE `departmentId` IN (SELECT `departmentId` FROM `department` WHERE `departmentName` = :i1b1)
```

###### In from

A sub query can replace the table. The alias is required, otherwise an exception is thrown.

```php
$older = (new Query())->setTable('user')->setCondition('age>', 40);

$records = $query->setTable($older, 'older')->get();
```

```sql
SELECT * FROM (SELECT * FROM `user` WHERE `age` > :i1b1) AS `older`
```

##### See the statement

`getSQL` returns the statement without running it. It does not clear the instance. By default values are put in place of the placeholders, which is handy for debugging.

```php
$query->setTable('user')->setCondition('age>', 40);

echo $query->getSQL();        # SELECT * FROM `user` WHERE `age` > 40
echo $query->getSQL(false);   # SELECT * FROM `user` WHERE `age` > :i0b1
```

**NOTE** The version with values is for reading only. Do not run it, because the values are not escaped. `getBinded` returns the bound values.

##### Run your own statement

`execute` runs any SQL you write. Bind values with `bindValue`, never by joining strings. It returns the rows as objects for a SELECT.

```php
$query = new Query();
$age = $query->bindValue(40);

$rows = $query->execute("SELECT firstName FROM user WHERE age > $age");
```

##### Transaction

Run several statements as one unit. All `Query` instances of a connection share the same transaction.

```php
$query = new Query();
$query->startTransaction();

try {
    (new Query())->setTable('account')
        ->setColumnWithValue('balance' . Query::AS_IT_IS, '`balance` - 100')
        ->setCondition('accountId', 1)
        ->update();

    (new Query())->setTable('account')
        ->setColumnWithValue('balance' . Query::AS_IT_IS, '`balance` + 100')
        ->setCondition('accountId', 2)
        ->update();

    $query->commit();
} catch (Throwable $e) {
    $query->rollback();
    throw $e;
}
```

##### Cache

`setCache` stores the result of `get` in cache. Pass your own key, or `true` to let Nishchay build the key from the statement. The second argument is the lifetime in seconds, where `0` means no expiry.

```php
$query->setTable('department')->setCache('departments', 3600);
$records = $query->get();

$query->isLastResultFromCache();   # true when the rows came from cache
$query->getLastCacheKey();         # 'departments'
```

The connection must be set up with a cache in [database settings](/learningCenter/settings/database), otherwise nothing is cached.

##### Quick reference

| Method | Purpose |
|---|---|
| `setTable` | Table, with optional alias or sub query |
| `setColumn` | Columns to select |
| `setCondition`, `setHaving` | Where and having |
| `addJoin` | Joins |
| `setGroupBy`, `setOrderBy`, `setLimit` | Grouping, sorting, limiting |
| `addUnionQuery` | Union |
| `get`, `getOne` | Select rows |
| `count`, `sum`, `max`, `min`, `average` | Aggregates |
| `setColumnWithValue`, `setColumnWithMultipleValue` | Values for insert and update |
| `insert`, `update`, `remove` | Change data |
| `quote`, `bindValue` | Build expressions safely |
| `getSQL`, `getBinded` | Inspect the statement |
| `execute` | Run your own SQL |
| `startTransaction`, `commit`, `rollback` | Transactions |
| `setCache` | Cache the result |
