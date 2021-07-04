#### Request Store

`Nishchay\Http\Request\RequestStore` class intended to store request data so that it can be used from anywhere. RequestStore has several static method using which we can add, update, remove and get data. These data are available for each stages of request and it gets destroy when request ends. Request store data is know by its name and name must be string.

Request store data are converted to variable by naming it with data name. All added data array available to view as variable.

###### Add
Method `add` always adds data. If data being added is already been added it throws exception.
```php
RequestStore::add('title', 'Nishchay');
```

To add multiple data at time, pass list of data in array where key should be name of data.

We can prevent exception which is thrown when data already exists by passing third parameter as `true`.

###### Get
Using `get` method we can access data stored in store. It returns `false` if data with the name not found in store.
```php
RequestStore::get('title');
```
It can return fallback value if data with the name not found in store, this can be done passing fallback value in second argument.
```php
RequestStore::get('title', 'Nishchay');
```

Above code will response with `Nishchay` if data with name `title` does not found in request store.

###### Update
Use `update` method to update request store data. If data not found, it returns false.

```php
RequestStore::update('title', 'Nishchay');
```

###### Set
`set` method adds data if it does not exist otherwise it updates it.
```php
RequestStore::set('title', 'Nishchay');
```
Above will add title to store if it is not already been added otherwise it updates it. We can pass array of data with key being data name to update or add multiple data at time. This method returns number of data update or added to request store.

###### Remove

Added data can also be removed using `remove` method. We can remove one data at time.
```php
RequestStore::remove('title');
```

If data to remove does not found in request store then this method returns `false`.

###### Get all data

Using `getAll` method we can get all added data. It returns array, key being data name.
```php
RequestStore::getAll();
```
###### Get type of data

Using `getType`, we can get type of data of given data name. It returns class name of data, if its an object. If data does not exists in request store then it returns `false`.
```php
RequestStore::getType('title');
```