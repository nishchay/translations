#### Database settings

Database connection and database related settings are located in `settings/configuration/database.php` setting file. Here we can defined multiple database connection, these connections are known by its database connection name as sepcified in settings.

Setting key `connections` contains list of database connections for our application. Where key presents its connection name. See below example how database connection for mysql can be created.
```php
'connections' => [
    'main_db' => [
        'type' => 'mysql',
        'dbname' => getEnvironment('DB_NAME', 'test'),
        'host' => getEnvironment('DB_HOST', '127.0.0.1'),
        'port' => getEnvironment('DB_PORT', 3306),
        'username' => getEnvironment('DB_USER', 'root'),
        'password' => getEnvironment('DB_PASS', 'root'),
    ]
]
```

Each conneciton should have following configurations:

| Name | Description |
|-----|-----|
| type | Supported database type. this can be `mysql`, `mssql` or `postgresql` |
| dbname | Database name |
| host | Host name or IP where database is located |
| port | Port number |
| username | Authentication username |
| password | Authentication password |


##### Set Default database connection

Using `default` config, we can specify default database conneciton for our application.

When we create instance of Query Builder or Database Manager and we do not pass database connection it will use default database connection. Same goes for Entity, if we do not define `@Connect` it will use default.

##### Restrict delete and update without condition.

This setting is applicable for query builder only. When this is `false`, it won't allow use to delete or update record without condition.

| Name | Description |
|-----|-----|
| updateNoCondition | Whether to allow update statement to be executed without condition. |
| removeNoCondition | Whether to allow delete statement to be executed without condition. |

##### Encryption
Encryption and decryption related configuration is contained within `encryption` second.

| Name | Description |
|-----|-----|
| type | `NULL` means use Nishchay encrption. `php` to sepcify your own callback function. `db` to use database side encryption, this means database side function will be used to encrypt and decrypt. |
| name | When type is `NULL`, name should be config name of encryption as denfined in `encryption.php` configuration |
| key | This is required when Nishchay encrption is not used. |
| callback | This setting divided into two parts `encrypt` and `decrypt` which have same setting name. First should be name of callback or database side encrypt and decrypt function name. |

When we use encryption other than nishchay, `callback` setting is used. Let's take a look at how database side function can be configured in this setting. So we will used database function `AES_ENCRYPT` and `AES_DECRYPT` to encrypt and decrypt values.

These two function are defined in `encrypt->function` and `decrypt->function` setting of `callback`. Nishchay will always pass value in the first argument and key in second argument so if there more parameter we need then we can specify in `parameters` setting as array. See below setting for database side encryption.

```php
/**
* Method to use for encryption.
*/
'encrypt' => [
    'function' => 'AES_ENCRYPT',
    'parameters' => []
],
/**
* Method to use for decryption
*/
'decrypt' => [
    'function' => 'AES_DECRYPT',
    'parameters' => []
]
```

Nishchay binds all parameters passed to this function.

If we want to php side callback function then it should be defined using `{class}::{method}` format only.

##### Global setting

Setting `global` section contains setting which act as default config for Entity class.

| Name | Description |
|-----|-----|
| derivedProperty | To set derived property while fetching and setting records values. |
| lazyProperty | To set derived object while fetching and setting record values. |
| refactoring |  Whether to refactor table column every time. For the first time you instantiate an entity, Nishchay refactors extra property in table. Nishchay creates this column in table if it does not exist. Nishchay also refactors Nishchay static data table in database when first instance of any entity is created. <br/><br/> `FALSE` means Nishchay thinks that table's extra property column exist in that table and static data table exist in database. |
| staticRefactoring | Whether to refactor static table when first instance of any entity created. `FALSE` means Nishchay thinks that static data table already been refactored.  `NULL` means it will use configuration value from `refactoring` config. |