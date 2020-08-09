#### Data - Query Builder

##### Introduction

Nishchay query builder provides various feature to create simple to complex query. Query builder remains same all supported databases.

##### Creating instance

`Query` class is located in `Nishchay\Data` namespace. Query accepts one optional parameter which should be name of database connection name. If we do not pass connection it choses default connection name from database setttings. Query instance created for one connection work for that connection only.
```php
$query = new Query('main');
```
Once instance is created it can be used for select, update, insert or delete records.

##### Select

###### All columns
```php
$records = $query->setTable('user')->get();
```
Above will fetch all records from _user_ table. `get` method fetch all matched records from table. Beause we did not applied any condition it selects all records.

###### Specific columns
```php
$query->setTable('user')
            ->setColumn(['userId', 'firstName', 'lastName'])
            ->get();
```
Above will fetch mentioned columns from user table. Column names should be passed as an array if want to fetch multiple column. If want to select only one column, pass column as string. Above code can also written as.
```php
$query->setTable('user')
            ->setColumn('userId')
            ->setColumn('firstName')
            ->setColumn('lastName')
            ->get();
```
Each time we call `setColumn` method, it adds all column names for selecting.

###### First row

`getOne` method returns first row from all matched records. This method applies LIMIT 1 to SELECT query.
```php
$query->setTable('user')
            ->setColumn(['userId', 'firstName', 'lastName'])
            ->getOne();
```
###### Column Alias

To create an alias of column name, just add key for column name.
```php
$query->setTable('user')
            ->setColumn(['userId', 'fname' => 'firstName', 'lname' => 'lastName'])
            ->get();
```
When we set key of any column name, it becomes alias of that column. This key name must be string. Nishchay checks for key type if its string it takes key as alias for column name.

###### Using function on column

Because function argument can contains mixed of column and values. It requires lot of processing to parse escape column name and bind value. Nishchay provides easiest way to use function in query.
```php
$query->setTable('summary')
        ->setColumn(['total'.Query::AS_IT_IS => SUM($query->quote('col1') .' + '. $query->quote('col2') . ' + ' . $query->bindValue(2).')']))
        ->get();
```
This will results in following query for mysql. Escaping of coulmns differs for each database connection.
```sql
SELECT SUM(`col1` + `col2` + :i0b1) as total;
```
In above php code `quote` method escapes passed value based on database connection. `bindValue` method binds value with unique placeholder name and returns it.

`Query::AS_IT_IS` applying after column alias(key) makes its value to use as it is. If there's no alias then `Query::AS_IT_IS` should be apply before expression.

It is advised to always use `quote` method on column name and `bindValue` for value if we are using expression.

##### Where

Using `setCondition` method we can apply where clause to query. It supports two parameter. First parameter can be string or array value. If we want to set one clause we can pass column name in first parameter and value in second parameter. When we need to set multiple clause, it can be achieived by passing array of clause in first parameter.

###### Simple
```php
$query->setCondition('firstName', 'TestName');
```
It will produce below statement clause.
```sql
`firstName` = :i0b2
```
Here second is considered as value and it will be bind.

###### Multiple clause
```php
$query->setCondition([
            'firstName' => 'TestName',
            'lastName' => 'Patel'
    ]);
```
Will produce below statement
```sql
`firstName` = :i0b3 AND `lastName` = :i0b4
```
When we pass array, then key should be column name or function.

###### Using operator

By defualt condition evaluates to equal clause. We can generate most type of clause. We just need to pass operator at end of left operand.
```php
$query->setCondition('age>', 18);
```
Generates following statement.
```sql
`age` > :i0b5
```
We can pass any operator at the end of left operand, query builder always checks for operator at the end of left operand. If no operator found at end, it use equal operator.

###### Matching column with another column
```php
$query->setCondition('point1' . Query::AS_COLUMN, 'point2');
```
When we pass `Query::AS_COLUMN` at end of left operand it treats value as column name and it produce below statement.
```sql
`point1` = `point2`
```
###### Use right operand as it is

If we need to use value as it is, we have to pass `Query::AS_IT_IS` at end of left operand.
```php
$query->setCondition('point1' . Query::AS_IT_IS, 'SUM(' . $query->quote('point2') . ' + 2)');
```
And this will result in following statement.
```sql
point1` = SUM(`point2` + 2)
```
###### Use left operand as it is

Left operand is always considered as column name, but sometimes we need to use function. In this case we have to use left operand as it is. This can be achieived by placing `Query::LEFT_AS_IT_IS` at end of left operand.
```php
$query->setCondition('SUM(' . $query->quote('point1') . ' + 2)=' . Query::LEFT_AS_IT_IS, 200);
```
will produce
```sql
SUM(`point2` + 2) = :i0b6
```

###### Operator precedence
When we use following constant on left operand and we also want to conditional operator other than equal then it should be place before these constants.

1. Query::AS_COLUMN
2. Query::AS_IT_IS
3. Query::LEFT_AS_IT_IS

```php
$query->setCondition('a>' . Query::AS_COLUMN,'b')
```
which generates
```sql
`a` > `b`
```
###### OR clause

By default multiple clause always result in AND clause, but it can changed to OR. This can be achieved by putting OR as element at last of array condition
```php
$query->setCondition([
        'firstName' => 'TestName',
        'lastName' => 'Patel',
        'age' => '28',
        'OR'
    ]);
```
When we put 'OR' at end of conditon array, query builder removes it and produces statement with OR cluase.
```sql
`firstName` = :i0b7 OR `lastName` = :i0b8 OR `age` = :i0b8
```
We don't have to do anything to produce AND clause as it can generated by default. But we cann still pass AND at end of condition array, just to denote that this condition will result in AND clause.

**NOTE:** Query builde does not check for validation of 'OR' and 'AND' at end of array. Query builder just apply whatever at end of array but only if only there's no string key for last element. Just for example below will generate statement like 
```sql
`firstName` = :i0b9 JOINER `lastName` = :i0b10 JOINER `age` = :i0b11
```

based on below code.

```php
$query->setCondition([
        'firstName' => 'TestName',
        'lastName' => 'Patel',
        'age' => '28',
        'JOINER'
    ]);
```
###### IN & NOT IN

To build IN clause, just pass array as a right operand and place `Query::IN` at end of left operand.
```php
$query->setCondition([
        'age' . Query::IN => [21,23,25]
    ]);
```
Query builder will bind each element of right operand and produces statement as shown below
```sql
`age` IN (:i0b12,:i0b13,:i0b14)
```
NOT IN clause can build by placing `Query::NOT_IN` at end of left operand.

###### Between
To produce between clause pass array having only two elements and left postfixed by `Query::BETWEEN`, see below code.

```php
$query->setCondition([
        'age' . Query::BETWEEN => [18,60]
    ]);
```

Above code results in following query statement

```sql
age BETWEEN :i0b15 AND :i0b16
```

###### Complex clause

```php
$query->setCondition([
        [
            'a' => 1,
            'b' => 2
        ],
        'c' => 3,
        [
            'd' => 1,
            'e' => 2,
            'f' => 3,
            'OR'
        ],
        'OR'
    ]);
```
Generates
```sql
(`a` = :i0b15 AND `b` = :i0b16) OR `c` = :i0b17 OR (`d` = :i0b18 OR `e` = :i0b19 OR `f` = :i0b20)
```
First element in condition results in AND clause as we did not have put anything at end for joining. If we do not specify key for element and directly place array, then it create seperate conditional group. Third element which results in OR clause as it have joining operator at end.

##### Join

Before adding join to query builder, we must need to setTable as its being used by join builder. `addJoin` allows creating join statement. It accepts array argument which has key as child table name and value of key should be string if both parent and child table need ot joined by same column name.

###### Two table having same column name
```php
$query->setTable('mainTable')
            ->addJoin([
                'childTable' => 'columnName'
            ]);
```
which results in below query.
```sql
SELECT * 
FROM `mainTable` 
LEFT JOIN `childTable` AS `childTable` ON `childTable`.`columnName` = `mainTable`.`columnName`
```
Here _childTable_ being the key represents child table name, value of it to produce ON clause for join. For above example we needed to join both child and parent table by same column name so we just used string.

###### Two table with different column name
```php
$query->setTable('mainTable')
            ->addJoin([
                'childTable' => ['childTableId'=>'mainTableId']
            ]);
```
When array is passed as join condition then by default each key(left operand) represents child table column and value(right operand) represents main table column. Here we have not specified table name for column, in this case query builder assigns table name as per default rule. _childTableId_ will be considered of _childTable_, _mainTableId_ will be considered of _mainTable_ table.
```sql
SELECT * 
FROM `mainTable` 
LEFT JOIN `childTable` AS `childTable` ON `childTable`.`childTableId` = `mainTable`.`mainTableId`
```
Join condition are same as we use in `setCondition` method. But here in join condition right operand is considered as column while for `setCondition` it is considered as value. To make right operand use as value place `Query::AS_VALUE` at end of left operand.
```php
$query->setTable('mainTable')
            ->addJoin([
                'childTable' => ['childTableId$'=> 23]
            ]);
```
**NOTE:**

*   Do not place `Query::AS_COLUMN` at end of left operand to denote right operand as column. This is also same for `setCondition` to not to place ``Query::AS_VALUE`` at end left operand to denote right operand as value. This is because, its default behavior so we don't need tell explicitly.
*   If you pass joiner at end of join condition, prefix it with `Query::ON_JOINER`. This is because in join condition joiner conflicts with join on same colum name. To better understand see below example
```php
$query->setTable('mainTable')
            ->addJoin([
                'childTable' => ['columnName', 'columnName1'=>'columnName2', Query::ON_JOINER . 'OR']
    ]);
```
If we do not prefix last element with `Query::ON_JOINER` in above code query builder will consider 'OR' as column name, because if we pass string its counted as column name. To solve this problem, it should be prefixed by `Query::ON_JOINER`. Query builder checks for last element in join condition which matches above condition. If it matches it is counted as joiner.

###### Joining more than one table
```php
$query->setTable('mainTable')
            ->addJoin([
                'childTable1' => 'columnName',
                'childTable2' => 'columnName'
            ]);
```
Builds following query
```sql
SELECT * 
FROM `mainTable` 
LEFT JOIN `childTable1` AS `childTable1` ON `childTable1`.`columnName` = `mainTable`.`columnName`
LEFT JOIN `childTable2` AS `childTable2` ON `childTable2`.`columnName` = `mainTable`.`columnName`
```
###### Join condiiton with other table name

Query builder adds main table and child table name on its own. It does only if we do not specify table name. Below example demonstrate how to use another table name.
```php
$query->setTable('mainTable')
            ->addJoin([
                'childTable1' => 'columnName',
                'childTable2' => ['columnName', 'columnName1'=>'childTable1.columnName2']
            ]);
```

```sql
SELECT * 
FROM `mainTable` 
LEFT JOIN `childTable1` AS `childTable1` ON `childTable1`.`columnName` = `mainTable`.`columnName`
LEFT JOIN `childTable2` AS `childTable2` ON `childTable2`.`columnName` = `mainTable`.`columnName` AND `childTable2`.`columnName1` = `childTable1`.`columnName2`
```
When we specify table name, query buider does not add table name. This applies to both left and right operand.

###### Specifying join type

It can be seen frmo above example that query builder by default generates LEFT join. It can be changed by specifying join type before table name.
```php
$query->setTable('mainTable')
            ->addJoin([
                Query::INNER_JOIN . 'childTable1' => 'columnName',
                'childTable2' => 'columnName'
            ]);
```
First table join will results in INNER join. Below are supported join types their join type.

*   Query::INNER_JOIN
*   Query::LEFT_JOIN
*   Query::RIGHT_JOIN
*   Query::CROSS_JOIN
*   Query::FULL_JOIN

###### Join table alias

Alias of join table is created by placing alias in opening and close bracket at end of table. Such example is listed below
```php
$query->setTable('mainTable')
            ->addJoin([
                'childTable(child)' => 'columnName'
        ]);
```

Above code prodices below statment

```sql
SELECT * FROM `mainTable`
LEFT JOIN `childTable` as `child` ON `child`.`columnName` = `mainTable`.`columnName`
```

##### Group By

Group by clause can be set by passing string if want to do grouping by single column or pass array if want to do grouping by multiple columns.

Group by single column.
```php
$query->setGroupBy('userId');
```
Group by multiple column.
```php
$query->setGroupBy(['userId', 'departmentId']);
```
##### Order By

By single column.
```php
$query->setOrderBy('userId');
```
By multiple column.
```php
$query->setOrderBy(['userId', 'departmentId']);
```
##### Having

Having clause can be set by `setHaving` and it works the same way as `setCondition`. To learn how to use, please read _where_ section of this doc.

##### Limit

To limit records to be returned, use `setLimit` method. This accepts two arguments, first being number of records to fetch and second to set offset from where record should start.
```php
$query->setLimit(5,10);
```
##### Insert

To insert record into table first set table name by using `setTable` method and then `setColumnWithValue` to set column name & its value. Then at last call insert method which will parepare insert statement and executes it.
```php
$query->setTable('user')
        ->setColumnWithValue([
            'firstName'=>'TestName',
            'lastName'=>'Patel',
            'age'=>28,
            'gender'=>'Male'
        ])
        ->insert();
```
Above can also be implemented by using chained method.
```php
$query->setTable('user')
            ->setColumnWithValue('firstName', 'TestName')
            ->setColumnWithValue('lastName', 'Patel')
            ->setColumnWithValue('age', 28)
            ->setColumnWithValue('gender', 'Male')
            ->insert();
```

###### Insert multiple records
To insert multiple records in one query use `setColumnWithMultipleValue` method. Here first parameter is list of column names and second parameter list of values. Once records to insert has been using this method then call `insert` method.

See below code:

```php
$query->setTable('content')
    ->setColumnWithMultipleValue(['contentKey','ContentValue'], [
        ['key1','Content 1'],
        ['key2','Content 2'],
        ['key3','Content 3'],
        ['key4','Content 4']
    ])
    ->insert();
```

List of values should be in same order as list of column names passed in first argument. If there's mismatch in count of row with count of column, query throws exception.

##### Update

Update works the same way as inserting record. We have to call `update` method at end so that query builder builds update statement and executes it.
```php
$query->setTable('user')
            ->setColumnWithValue('firstName', 'TestName')
            ->setColumnWithValue('lastName', 'Patel')
            ->setColumnWithValue('age', 28)
            ->setColumnWithValue('gender', 'Male')
            ->setCondition('userId', 1)
            ->update();
```
Just like setting condition we can also tell wheather value is column or expression by applying same symbol at end of columnName. Put at end of column name `Query::AS_COLUMN` to denote value is column name and `Query::AS_IT_IS` to use value as it is.

###### Update by JOIN
We can also update table by using join. For that just add join and mention what should be updated in `setColumnWithValue`.
```php
$query->setTable('mainTable')
            ->addJoin([
                'childTable'=>'columnName'
            ])
            ->setColumnWithValue('childTable.columnName', 'columnValue')
            ->setCondition('mainTable.columnName', 1);
```
##### Remove

To delete records just set condition and then execute `remove`.
```php
$query->setTable('user')
            ->setCondition('user.userId', 1)
            ->remove();
```

###### Remove by JOIN

Just like update, record can also be deleted by using join.
```php
$query->setTable('mainTable')
            ->addJoin([
                'childTable'=>'columnName'
            ])
            ->setCondition('mainTable.col', 1)
            ->remove();
```
##### Sub Query
We can also add subquery in `from`, `column`, `where`  clause and in `join` on clause. We just have to create an instnace of `Query` class, prepare query and add it to main query builder instance.

###### In column
While selecting column, we pass array consisting of key and value. When key is string it becomes alias and value becomes column name. If we put query instance in value, it becomes subquery. 

For subquery in column name, alias is required.

```php
$query->setTable('mainTable')
            ->setColumn(['columnName' => (new Query())->setTable('subTable')->setCondition('subColumnName#','mainTable.columnName')])
            ->get();
```
It will build query as shown below.
```sql
SELECT (SELECT * 
FROM `subTable`  
WHERE `subColumnName` = `mainTable`.`columnName`) AS `columnName` 
FROM `mainTable`
```
###### In where clause
Just like how subquery is used in selecting column, we can also pass subquery as right operand of where clause.
```php
$query->setTable('mainTable')
            ->setCondition([
                'mainTable.columnName' => (new Query())->setTable('subTable')->setCondition('subTableColumnName' . Query::AS_COLUMN,'mainTable.columnName')
            ])
            ->get();
```
Which will result in following query.
```sql
SELECT * 
FROM `mainTable`  
WHERE `mainTable`.`columnName` = ((SELECT * 
FROM `subTable`  
WHERE `subTableColumnName` = `mainTable`.`columnName`))
```
Above code will create equal condition for clause which is default behaviour, but it can changed to IN, NOT IN and it also supprts conditional operator. Below code is updated, so that IN conditon can be created.
```php
$query->setTable('mainTable')
            ->setCondition([
                'mainTable.columnName'. Query::IN => (new Query())->setTable('subTable')->setCondition('subTableColumnName' . Query::AS_COLUMN,'mainTable.columnName')
            ])
            ->get();
```
###### In from table

Just like in column and where condition, subquery can also be added to from table. When we use sub query for `from` table, alias is requured.
```php
$query->setTable((new Query())->setTable('mainTable')
            ->addJoin(['childTable'=>'columnName'])
            ->setCondition('childTable.columnName2'. Query::AS_COLUMN,'mainTable.columnName2')
            ,'mainTable');
```
Which will result in below query statement

```sql
SELECT * 
FROM (
    SELECT * 
    FROM `mainTable` 
    LEFT JOIN `childTable` ON `childTable`.`columnName` = `mainTable`.`columnName` 
    WHERE `childTable`.`columnName2` = `mainTable`.`columnName2`
) AS `mainTable`
```