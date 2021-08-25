#### Database Manager

##### Introduction

Database Manager provides features to create or alter table by adding, updating & removing column, primary and foreign key. Just like Query builder database manager builder builds query based on database type. Database manipulation builder remains same for all supported database type.

##### Creating instance

```php
$db = new Mishchay\Data\DatabaseManager();
```

`DatabaseManager` also have one optional connection name, if not passed then default database connection used.

##### Create or Alter table

Whether to create or alter table is auto detected by database manager. When we set table name, database manager checks if table with name already exist or not. If table already exist, builder builds query for altering database table.

```php
$db->setTableName('user')
```

##### Column

###### Add or Change

Add or change column decided based on how we pass column name. To add column pass column name as string or pass an array with key as existing column name and value as new column name. When we want to change column's data type or any other thing, then both key and value should be same name.

Column can be added or changed by using `setColumn` method. Where its all parameter are explained below:
| Property | Type |
| ----------- | ----------- |
| name | Column name to add or change. Pass table name as string to add. To change existing column pass in array, key as old column and its value as new column name. When don't want change column name pass key and value same |
| dataType | Data type of column. This should be string if data type does not need length argument. Otherwise it should be pass as `['dataTypeName','length']` |
| default | When we omit this argument or pass null then `DEFAULT NULL` is applied. To make column `NOT NULL` pass `false`. Other value is counted as default value for column. |
| comment | Comment for column |

**NOTE:** Changing column requires their original datatype, default and comment to be passed in every time.

Creating new column named `id` and its data type `int`

```php
$db->setColumn('id', 'INT');
```

Datatype which with its length.

```php
$db->setColumn('firstName', ['varchar', 50]);
```

NOT NULL Column

```php
$db->setColumn('firstName', ['varchar', 50], false);
```

Default null column, omit third argument or pass null.

```php
$db->setColumn('firstName', ['varchar', 50], null);
```

Default value.

```php
$db->setColumn('profilePicture', ['varchar', 200], 'images/defaultPicture.jpg');
```

Applying column comment

```php
$db->setColumn('firstName', ['varchar', 50],null,'Comment sample');
```

Change column name

```php
$db->setColumn(['firstName' => 'userFirstName'], ['varchar', 50]);
```

Change column data type or data type length

```php
$db->setColumn(['firstName' => 'firstName'], ['varchar', 250]);
```

In above code because we are passing key and value same, column name won't change

###### Remove column

To remove one or more colum from table use `removeColumn` method. If want to remove only one column, pass column name as string. For the multiple columns to be remove, pass list of columns in an array.

```php
# Remove single column
$db->removeColumn('columnName');

# Remove multiple column
$db->removeColumn(['name1', 'name2']);

# Or by using chained method
$db->removeColumn('name1')
    ->removeColumn('name2');
```

##### Constraints

###### Primary key

Primary key is added by using `setPrimaryKey` method. We only need to pass column name on which primary key need to be created.

```php
    $db->setPrimaryKey('id');
```

###### Foreign key

Foreign key is added by using `addForeignKey` method. We can call this method multiple times to add more then one foreign key. This method have following parameters.

| Property     | Type                                                                                                                                                                            |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| selfColumn   | Name of column on which foreign need to be created                                                                                                                              |
| parentColumn | Parent column name to which self column will be referenced to. This should be an array with following details `['table' => 'parentNameTable', 'column' => 'parentColumnName']`. |
| onUpdate     | Optional, default is CASCADE. What should happen on updating, CASCADE, RESTRICT or else.                                                                                        |
| onDelete     | Optional, default is CASCADE. What should happen on deleting, CASCADE, RESTRICT or else.                                                                                        |

```php
    $db->setTable('address')
        ->addForeignKey('userId',['table' => 'user','column' => 'userId'],'CASCADE', 'CASCADE');
```

Above will add foreign key to column _userId_ of _address_ table which reference to _user.userId_

###### Index key

Index key is added by using `addIndexKey` method which have following parameter.

| Property | Type                                            |
| -------- | ----------------------------------------------- |
| column   | Column name on which index need to be created.  |
| type     | Type of index, which is based on database type. |

We do not need to create index key, We only have to give column name and type of index and then database manager builder creates index key with following format. `IDX_{tableName}_{columnName}`. Below example show how FULLTEXT index can be added.

```php
$db->setTable('user')
->addIndexKey('firstName', 'FULLTEXT');
```

##### Remove Constraint

Primary Key, Foreign Key and Indexes are removed using `removeConstraint` method. Constraint can be removed by specifying column and type of constraint or by specifying constraint key. This method has three parameter which are listed below:

| Property | Type                                                                                                                                   |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| name     | Column name from which constraint need to be removed. If third parameter is false then this should be key of constraint                |
| type     | Type constraint, this should be from _primary, foreign or index_.                                                                      |
| key      | Optional parameter. Pass true if passing constraint key in first parameter. This is to denote if first parameter is key or column name |

```php
    # Remove primary key.
    $db->setTable('address')
        ->removeConstraint('addressId', 'primary');

    # Remove foreign key.
    $db->setTable('address')
        ->removeConstraint('userId', 'foreign');

    # Remove index key.
    $db->setTable('address')
        ->removeConstraint('landmark', 'index');
```
