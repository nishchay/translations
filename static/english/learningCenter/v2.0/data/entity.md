#### Data - Entity Manager

##### Introduction

Entity is class presentation of database table where class name becomes table name and its property maps to table columns. Once entity class is created for table, it can be used to manage data stored within that table. It allows selecting data from table or reflecting changes in table that is update, insert or remove.

We don't have to worry about creating query for selecting or managing data. Because Nishchay data processor prepares query based on database type of your class. This makes entity class to work with all supported database type, there won't be changes to your class even if you change database for it.

To select or manage data of entity, it requires EntityManager. `Nishchay\Data\EntityManager` instance is created with entity class.

Let's start with creating our first simple entity class for table.

1.  We will create entity for _User_ table.
2.  Table user has following columns:

| Property  | Type   | Identity |
| --------- | ------ | -------- |
| userId    | Int    | Yes      |
| firstName | string | -        |
| lastName  | string | -        |
| gender    | string | -        |
| birthDate | date   | -        |

##### Entity[time:120]

To make class an entity, declare `Nishchay\Attributes\Entity\Entity` attribute on class. Below example demonstrates how to declare `Entity` attribute on class.

```php
use Nishchay\Attributes\Entity\Entity;

#[Entity(name:'user')]
class User {
```

This class will be map to 'user' table in database.

Parameters of `Entity` attribute is listed below.

| Parameter | Description                                                                                                                                                                                                                                                            |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name      | Entity class will be mapped to given name. If value is _this_, then table name will be same as name of full class name. To consider base class name as table then value should be _this.base_                                                                          |
| separator | If the table name contains forward slash it will be converted to passed separator. This happens only when we user full class name as table name. We can use any value for separator, but be careful it should not results into error by using inappropriate separator. |
| case      | Whatever is table name, we can convert table name to lowercase, uppercase or use same as defined. By default case is same, it will not change case of table name.                                                                                                      |

Below code demonstrates various usage of above three parameter. To explain, we are taking class name as _Application\Entity\User_

```php

// Table name will be 'user'.
#[Entity(name:'user')]

// Table name will be 'Application_Entity_User'
#[Entity(name:'this')]

// Table name: ApplicationEntityUser
#[Entity(name:'this',separator:'')]

// Table name: applicationentityuser
#[Entity(name:'this',separator:'',case:'small')]

// Table name: User
#[Entity(name:'this.base')]

// Table name: USER
#[Entity(name:'this.base',case:'upper')]
```

##### Connection

An application can connect multiple database. By default entity uses default database connection as specified in config. But Nishchay also allow us to use other connection than defined in database config by using `Nishchay\Attributes\Entity\Connect` attribute.

`Connect` supports only one parameter called _name_ whose value should connection name. Below example demonstrates how we can use `Connect` attribute on entity class.

```php

#[Entity(name:'this.base',case:'small')]
#[Connect(name:'auth')]
class Auth {
```

##### Property

Each property of entity class maps to database table column if it matches following:

1.  `Nishchay\Attributes\Entity\Property\DataType` or `Nishchay\Attributes\Entity\Property\Derived` declared on property.
2.  Property name should not start with underscore.

Remember following points too:

- Access type of property does not matter.
- Property declared as private or protected are still accessible from entity manager instance.
- Property name should be same as table column.

Mapping our first column to property.

```php
use Nishchay\Attributes\Entity\Entity;
use Nishchay\Attributes\Entity\Property\DataType;

#[Entity(name:'user')]
class User {

    #[DataType(type:'int')]
    public $id;

}
```

When we declare `DataType` attribute on property, that property will be mapped to table. Here _id_ will be mapped to _id_ column of _user_ table.

Below code demonstrates how whole table can be created as entity.

```php

#[Entity(name:'user')]
class User {

    #[Identity]
    #[DataType(type:'int',readOnly:true)]
    public $id;

    #[DataType(type:'string', length:20, required:true)]
    public $firstName;

    #[DataType(type:'string', length:20, required:true)]
    public $lastName;

    #[DataType(type:'string', length:10, required:true)]
    public $gender;

    #[DataType(type:'date', required:true)]
    public $birthDate;
}
```

##### Create an instance of Entity Manager

Creating an instance of Entity Manager class is very easy.

```php
$user = new EntityManager(User::class);
```

Entity Manager accepts only one parameter which should be class name or an instance of Entity Class.

Now we have _$user_ using which we can select or manipulate data. Now and on we will use _$user_ to demonstrate most example.

##### Fetch single record

`get` method of an Entity Manager class allows us to fetch single record from table using its primary(identity) key.

```php
$record = $user->get(1);
```

`get` method returns instance of self class but this behaves exactly like instance of an User class. We can access each property of class even if its defined as private or protected. We can also call method which are public.

##### Fetch all records

`getAll` methods returns all records from table. It will return array of Entity Manager instance representing each row.

```php
$records = $user->getAll();
```

##### Access/Update property

Property of entity class can be accessed the same way as it is property of EntityManager class. Suppose we want to print id of an user, it can be printed just with following syntax.

```php
<?= $record->id; ?>
```

Let's change first name of an user.

```php
<?php $record->firstName = 'Ray' ?>
```

Above will update first name in entity class only, but it won't be reflected in table. To learn how to reflect changes into table, please read below section.

##### Data manipulation

Any changes made to entity property does not directly reflected to table. There are number of methods using which we can decide how it should be reflected into table.

###### Insert

Once we create an instance of Entity Manager class for Entity. We can directly use it to set value of each properties, then calling `insert` method on same instance will insert record into table.

```php
$user->firstName = 'Bhavik';
$user->lastName = 'Patel';
$user->gender = 'male';
$user->birthDate = DateTime::createFromFormat('Y-m-d', '1991-01-01');
$user->insert();
```

Above will insert record into table. If you have fetched record and call insert method on it, it will still insert record.

###### Update

`Update` works on only fetched record. Calling it on instance which was not fetched, will not do anything.

```php
$record = $user->get(1);
$record->firstName = 'Ray';
$record->update();
```

###### Save

Calling insert method will always insert record and update will always update record. If we call insert multiple times on same instance it will always insert record. We have `save` method which works smartly. It will insert record if record is not fetched from database, then calling save method again on it will update its value. If the record is fetched then it will do update.

Save method first checks if its need to be inserted or updated, then it takes appropriate action. If no changes has been made to instance then nothing happens. Not even query will be fired.

###### Remove

Just like `update` method `remove` works on fetched record only. Calling remove method will remove record from table.

```php
$record = $user->get(1)->remove();
```

##### Setter methods - Update value before it returns

Setter methods are called on property before it returns but after applying value to entity class. Suppose we want make sure that first name of an user should be first case upper and remaining lower then we can create setter method to do same.

Create setter for firstName property.

```php
public function setfirstName($firstName){
    return ucfirst(strtolower($firstName));
}
```

Setter method follows this pattern `set{property_name}`. Here property name's first character should be in upper case and remaining unchanged. Value returned by setter method get assigned to entity class property.

##### Data Type of property

Entity supports various types of data including custom.

###### Integer

Entity property supports integer data type. How many types of integer can support property is depends on entity's database environment. Suppose for MySQL, it can support TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT. For integer data type value of _type_ parameter of `DataType` attribute should be _int_ and this remains same for all database environment. Based on length of data type we set in parameter, which data type from database environment to choose is decided. For integer data type value should be byte. As in MySQL uses 4 byte for INT data type, setting _length: 4_, will choose INT data type.

```php

#[DataType(type:'int',length:4)]
public $userId;
```

###### Float/Double

Property which has fractional point. Float and Double data type can be use for property.

```php

#[DataType(type:'float',length:5.2)]
public $amount;
```

Above will allow property to store 5 digit value of which 2 can be fractional point.

###### Date/DateTime

If the property type is Date or DateTime then its value should be instance of DateTime class. Fetched value from table converted to DateTime instance by entity manager. Upon reflecting to database table Nishchay converts it into type appropriate to database environment.

```php
#[DataType(type:'datetime')]
public $createdAt;
```

###### Array

This is just like php array. Value is serialized while reflecting into database and unserialized upon assigning it to property.

###### Mixed

Value type mixed means it can support any scaler value except Date/DateTime as it is class.

###### Custom

We can specify class name to type. When we give class name to property it is serialized while reflecting into database and unserialized upon assigning it to property.

##### Property attributes

###### Identity

Each class should contain identity property. Only one identity property per class is allowed. To make property an identity declare `Identity` attribute on it.

```php

#[Identity]
public $id;
```

###### Required

Property can be make required. Property which are required, its value should not be null. To make any property required, set required parameter value of `DataType` attribute to _true_.

```php

#[DataType(type:'string', required:true)]
public $firstName;
```

###### Read only

If the property is an read only, its value can not be changed. Its wise to make identity property read only. Once we set value to read only property, it can not changed later. We can make any property read only.

Readonly property can only be set for new record only. If it is fetched from database then it can not be changed.

```php

#[Identity]
#[DataType(type:'int', readOnly:true)]
public $id;
```

We can also use readonly property which should not be changed once its saved in database on insert.

###### Allow values from list only[time:60]

We can define list of values and property value can only contain value from defined list ony.

```php

#[DataType(type:'string', values=['male','female'])]
public $gender;
```

By defining above `values` parameter on above property, we are restricting property to have only _male_ or _female_ only.

```php
$user->gender = 'invalid';
```

Above code will throw exception because value being assigned is not from list.

###### Default value[time:120]

Default value applicable for new record only but it should not be for required property. If we are inserting a record while property is (means setting property to null) untouched and it has default value defined for data type, then default value will be applied to property.

If record has been fetched from database and setting its value to null then default value won't be applied.

Below is an example for how to define default value for property

```php

#[DataType(type:'int', default:1)]
public $userTypeId;
```

Below code will use default value _1_ for _userTypeId_ property.

```php
$user = new EntityManger(User::class);
$user->fullName = 'Bhavik Patel';
$user->gender = 'Male';
$user->save();
```

Because we are not setting any value to property _userTypeId_ will have default value _1_.

Default value can be dynamic for `Date` and `DateTime` data type.

If an entity contains properties like `createdAt` which will have time when the record was inserted. Below example will make property to have default value as current time.

```php

#[DataType(type:'datetime', default:'now')]
public $createdAt;
```

Setting default value `now` for the `Date` or `DateTime` data type will insert current time.

For these data type, `default` can have any value which can modify date. Check [here](https://www.php.net/manual/en/datetime.formats.relative.php) for supported values.

###### Static property

Entity can also have static properties. There's no special attribute to define static property. Entity static property works same way as static property of standard class.

Static properties are accessed directly using its class name. Before we access static property it needs to be initialized only once. As soon as we create first instance of Entity Manager class, all static properties are initialized and this happen only once. Entity Manager have separate method to save static properties in database.

###### Update static property

To reflect changes in database, call `saveStatic` method on instance of Entity Manager. Suppose we have created instance of Entity Manager for User entity class and we call `saveStatic` it reflect static property of User class only. If there are no static property in entity class and we call `saveStatic` nothing will happen.

`saveStatic` method first checks if there are any changes in any static property. Only property which are updated will be reflected to database, nothing happens for unchanged properties.

###### Relative[time:60]

This works like foreign key of table. Property can be relative to identity property or property of another class. If we want to relate one entity to another entity, we can do that with the help of relative property.

Relative property is defined using `Nishchay\Attributes\Entity\Property\Relative` attribute. See below example.

```php

#[Relative(to:Department::class)]
#[DataType(type:'int')]
public $departmentId;
```

Here _departmentId_ will be relative to identity property of `Application\Entity\Department` class.

Value of _to_ should be full class name.

###### Relate property other than identity[time:60]

If we need to relate property to property other then identity property, it can be done with the help of _name_ parameter. Value of _name_ should be property of _to_ class. This is optional parameter, if we don't define this parameter then identity property of relative class is considered for this parameter.

```php

#[Relative(to:Department::class, name:'departmentId')]
#[DataType(type:'int')]
public $departmentId;
```

###### Relative loos or perfect[time:60]

We can define relativity relation type with the help of `type` parameter of `Relative` attribute. Value of this parameter should be _loose_ or _perfect_.

If `type` is perfect, then value of property should exists in relative class. Leaving property null or assigning value which doest not exists in relative class results in an exception.

```php

#[Relative(to:Department::class, type:'perfect')]
#[DataType(type:'int')]
public $departmentId;
```

We can make this property loosely relative to relative class by assigning value _loose_ to `type` parameter.
Assigning null to this property won't result in an exception, but assigning value which does not exists in relative class will still result in an exception.

###### Derived[time:120]

Derived properties are property whose value is derived from another entity or it can be generated from callback. `Nishchay\Attributes\Entity\Property\Derived` attribute makes property derivable, it has many parameter and are listed below.

| Property | Description                                                                                                                                          |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| from     | should be property name of self class which should be relative to another entity or class name which has property relative to identity of self class |
| join     | should be callback method name which returns join statement, callback response will be assigned to property                                          |
| callback | should be callback method name, callback response will be assigned to property                                                                       |
| property | list of properties whose value needs to be derived.                                                                                                  |
| group    | to apply group by clause                                                                                                                             |
| hold     | whether derived value should be single or multiple                                                                                                   |
| type     | relativity loose or perfect                                                                                                                          |

We do not have to declare `DataType` attribute on derived property as its datatype will be the same as property we fetch from another entity.

**NOTE**: We can not change value of derived property because deriving property is not owned by self class. Class can reflect changes to database only of its own class.

###### Deriving based on relative property of self class

If one of property in entity class is relative to another entity(related entity), then we can fetch related entity.

```php

#[Relative(to:Department::class)]
#[DataType(type:'int')]
public $departmentId;

#[Derived(from:'departmentId')]
public $department;
```

Because we have specified that value of property _$department_ should be fetched by _$departmentId_ property. Entity manager will fetch record with value of _$departmentId_ and creates an instance Entity instance for `Application\Entity\Department`, then instance of this class will be assigned to _$departmentId_.

###### Deriving specific property

By default Entity manger fetches whole entity for the derived property. But in some case we might not need all properties of related class, in this case we can use `property` parameter to specify one or more property which only need to be fetched.

`property` parameter of `Derived` attribute accepts string or array. If we have to derive more than one property, specify each properties in an array.

```php

#[Relative(to:Department::class)]
#[DataType(type:'int')]
public $departmentId;

#[Derived(from:'departmentId', property:['departmentName', 'departmentCode'])]
public $department;
```

Entity Manager will now fetch only two properties from related class and both of these properties will be assigned to _$department_ property.

###### Form class which has property relative to identity of self class[time:180]

To better understand this case, let's take one example. We have User entity which stores all user data. These users have different skills which are stored in another entity. Skill entity stores `userId` and `skillName` and identity of this class `skillId`.

Now when we fetch user record, we also want to fetch all skills of fetched user. To do that we will create one derived property called `skills` which will be relative to `Skill` entity.

When we specify class name instead of relative property name of self class, Entity Manager will look for property which is relative to self(`User`) entity in related entity (`Skill`). Based on this record will be fetched for derived property.

Skill Entity

```php

 #[Identity]
 #[DataType(type:'int', readOnly:true)]
public $skillId;


 #[Relative(to:User::class)]
 #[DataType(type:'int')]
public $userId;


 #[DataType(type:'string',length:50)]
public $skillName;
```

User Entity

```php

 #[Identity]
 #[DataType(type:'int', readOnly:true)]
public $userId;

 #[Derived(from:Skill::class, hold:'multiple')]
public $skills;
```

###### Derive multiple records from related entity

By default Entity Manager fetches only one record for the entity. In some case we might need multiple records from related property.

Such example is mentioned in above section, where user can have multiple skills. If you look at how `Derived` attribute is declared on _$skills_ property, we have defined _hold_ parameter.

Using `hold` parameter we can tell Entity Manager to fetch multiple records instead of single record.

```php

 #[Derived(from:Skill::class, hold:'multiple')]
public $skills;
```

###### Deriving based on extended relation[time:300]

We can also derive entity or entity property which have extended relation. To explain this, we will take example of user only. User can belongs to department and this department can have Zone. This Zone stores zone name and address.

When we fetch user record, we also want to fetch zone detail of department which user is belongs to. As `Department` stores on _zoneId_, fetching `Department` entity only we won't have zone detail. Before we go further see entity properties of `Department` and `Zone`.

Department entity.

```php

 #[Identity]
 #[DataType(type: 'int', readOnly: true)]
public $departmentId;

#[DataType(type: 'string', length: 50, required: true)]
public $departmentName;

#[Relative(to:Zone::class)]
#[DataType(type: 'int', required: true)]
public $zoneId;
```

Zone entity

```php

 #[Identity]
 #[DataType(type: 'int', readOnly: true)]
public $zoneId;

#[DataType(type: 'string', length: 250, required: true)]
public $zoneName;

#[DataType(type: 'string', length: 50, required: true)]
public $zoneAddress;
```

To achieve this we specify extended relation separated by dots. Let's take a look how `User` entity can be modified to fetch zone.

```php

 #[Identity]
 #[DataType(type: 'int', readOnly: true)]
public $userId;

#[Relative(to:Department::class)]
#[DataType(type: 'int', required: true)]
public $departmentId;

#[Derived(from: 'departmentId.zoneId')]
public $zone;
```

If you are not able to spot, take a look at `from` parameter of `Derived` attribute on `$zone` property of above class.

This will fetch whole `Zone` entity but we can define `property` parameter to fetch only specified properties.

We can also derive values using class separated dots.

###### Specifying how records should be fetched: Applicable for _from_ type only

Records are fetched based on perfect or loose relationship. For the perfect relationship, record should exists in relative class. If record does not exists in relative class, no record will be returned. While for the loose relationship, if there's no record in relative then null value is assigned to property.

If we do not define type relationship using `type` parameter on derived property then relationship is taken from relative property. If relative property does not have `type` defined then perfect relationship applied by default.

```php

#[Relative(to:Department::class, type: 'loose')]
#[DataType(type: 'int')]
public $departmentId;

#[Derived(from:'departmentId', property: ['departmentName', 'departmentCode'])]
public $department;
```

For above code loose relationship is applied because _$department_ does not have defined such relationship and relationship has been taken from _$departmentId_ property.

Suppose _$departmentId_ does not have `type` parameter set then default `perfect` would be used.

###### Derive using custom join

Deriving using `from` parameter is for straightforward relationship and is perfect relationship which follows standard.
If we are not able to derive using `from` parameter, we can use `join` parameter, which should have callback method name declared in same class.

This callback method must return join rule. Callback method must exists in same class. As joins returned by callback method can have multiple table, we have to tell Entity Manager which property to fetch from which Entity.

_NOTE_: `from` parameter also limits deriving property from one entity only, while using join callback we can derive property from multiple entity for derived property.

Example explained in above section can also achieved using join callback.

```php

 #[Derived(join: 'getZoneJoin', property: ['zone.zoneName', 'zone.zoneId'])]
public $zone;

public function getZoneJoin()
{
    return [
        '[<]' . Department::class . '(department)' => 'departmentId',
        '[<]' . Zone::class . '(zone)' => ['zoneId' => 'department.zoneId']
    ];
}
```

Please learn [here](/learningCenter/data/entityQuery), for how join works for Entity class.

###### Using callback

For properties whose value is prepared based on fetched record, can be created using callback method. Response of callback method is assigned to property.

Using `callback` parameter we can specify callback method for the derived property.

Because callback exists in your entity class, you can access each property's value by using `$this`. You might not be able to access some derived property which is based on callback as order of processing callback property depends on how its defined in class.

```php

 #[Derived(callback: 'prepareFullname')]
public $fullname;

public function prepareFullname()
{
    return $this->firstName . ' ' . $this->lastName;
}
```

**NOTE:** For callback derived properties, all other parameters are ignored.

###### Fetching array of values

By default derived property fetch only first value, but sometimes we might need to fetch array of values form another entity. Parameter `hold` can accepts _single_ or _multiple_, if we do not define it then by default _single_ is record is fetched. Setting _multiple_ fetches multiple values.

Just like fetching whole entity, fetching array of values makes derived property lazy. Read more about [Lazy properties](#lazyProperties).

###### Applying group clause

We sometime might need group clause for record which needs to be fetched. Such thing can be applied using `group` parameter. Whatever you specify in `group` parameter is taken in group by clause.

##### Lazy properties

If property is holding array of values or whole entity, it becomes lazy. Separate query is fired to fetch whole entity. By default lazy properties are disabled, we need to enable it to fetch lazy properties. We can do that with the help of `enableLazy` and `enableDerived` method, here parameter must be boolean.

Use `enableLazy` to enable properties which hold array of values and `enableDerived` to enable properties which holds whole entity.

Suppose we have one derived property which hold whole entity and we have enabled lazy property then two query will be fired. One for fetching entity records and other for fetching deriving entity records. Derived properties gets value after property which has value of self entity. Same thing happens for property which holds array of values.

For each entity property gets value in following order.

1.  Property which has value from self entity
2.  Derived property which holds whole entity or array of value but only if lazy property is enabled.
3.  Callback derived property

Lazy and derived properties can enabled globally by changing value in `database` config `global`.

##### Property value validation

Based on attribute `DataType`, `Derived` and `Relative` declared on property, Entity Manager validates record upon fetching or while updating property value.

Such example, _User_ entity's identity property is of data type `int`. Assigning string value to this property results in an error. Same thing is applied for each data type. It also validates if property is readonly or not, its relativity with another entity, whether it is required.

Apart from this validations we can also define our own validation for property. This can be achieved by declaring `Validation` attribute on property. This attribute accepts only parameter called `callback` which should have callback method name.

If callback exists in same then specify only method name otherwise specify in `{class}::{method}` format.

Validation callback must return boolean. Returning value other than `true` invalidates property value and prevent it from assigning value or reflecting changes to database.

```php

#[Validation(callback: 'validateAge')]
#[DataType(type: 'int')]
public $age;

public function validateAge($age) {
    return $age >= 18
}
```

Above will allow age property to have user's age greater or equal to 18 only.

##### Extra property

All entity records have properties defined within that class. Apart those property, we can also add more properties to row which is called extra property for the row. Extra property added for one row is visible for that row only.

This extra property behave same as property defined within class. It can be accessed and update same as class property.

To create an extra property, we first need to create extra property using `ExtraProperty` class. We must bind it to row to which entity should be applied to.

`ExtraProperty` have following method which can be used to set data type, required and readonly attributes.

| Method      | Description                                  |
| ----------- | -------------------------------------------- |
| setDataType | Sets data type of the property.              |
| setLength   | Sets length of data type.                    |
| setReadOnly | Makes property read only                     |
| setRequired | Makes property required                      |
| bindTo      | Row to which extra property need to be added |

Once we prepare extra property, we need DataClass Reflection for the Entity. Using which we can add property to row.

```php
$entity = new EntityManager(User::class);

# Fetching single record
$row = $entity->get(1);

# Preparing extra property
$extra = new ExtraProperty('extra');
$extra->setDataType('int')
        ->setRequired(true)
        ->bindTo($row);

# Creating reflection class so we can add extra property to row.
$reflection = new DataClass(User::class);
$reflection->addExtraProperty($extra);

# Once extra property is added to row, we can then set its value.
$row->extra = 3;
```

Above will add extra property to `$row`. Next time we fetch `$entity->get(1)`, it will come with extra property called _extra_

##### Temporary data

As the name suggest we can also add temporary data to row if needed, it gets removed along with object. Temporary data are just stored in memory and are not reflected to database. Its get lost once object destroyed.

We can add any kind of temporary data, there are no limitation and validation on it. Once temporary data added it can also be removed. Below are some methods to manipulate temporary data.

| Method       | Description                                                                      |
| ------------ | -------------------------------------------------------------------------------- |
| setTemp      | which accepts two parameter, first temporary data name and second for its value. |
| getTemp      | Returns temp data if it exists otherwise it throws exception                     |
| isTempExists | Returns TRUE if data with name exists in temp data                               |
| removeTemp   | Returns TRUE if data with name removed from temp                                 |

##### Events

Events are executed on specific action. Action can be insert, update or remove. There are two types of events, one is execute before(`Nishchay\Attributes\Entity\Event\BeforeChange`) any action is taken place and other is executed after(`Nishchay\Attributes\Entity\Event\AfterChange`) the action has been taken place.

`AfterChange` and `BeforeChange` both have same parameter which are listed below:
| Parameter | Description |
| ----------- | ----------- |
| callback | Can be method name of same class or it can be in `{class}::{method}` format. If annotation is defined on method then this parameter will be discarded. |
| priority | Priority of events |
| for | Event for which action(insert, update, remove). By default action event will be considered for all actions |

Both event attributes can be declared on entity class or method. When annotation defined on method, callback parameter will be discarded.

**NOTE:** Unlike other attribute `BeforeChange` and `AfterChange` can be defined multiple times.

Event callback must return boolean value. Value other than `true` considered as failure of event, applicable for before change event only. Exception is thrown and operations is discarded if callback returns failure.

Event callback receives following parameters.
| Parameter | Description |
| ----------- | ----------- |
| old | All properties having old values |
| new | All properties having new values |
| mode | Mode for which event has been called. This can be any of `insert`, `update` and `remove` |
| updatedName | List of properties whose value has been changed |

_We can't change `old` and `new` values in events._
