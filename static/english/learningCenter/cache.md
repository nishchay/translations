#### Cache

Nishchay supports two caching servers memcached and redisdb. To make caching work caching server and its extensions must be installed.

Settings related to cache are contained in `settings/configuration/cache.php` file. 

###### Cache instance

Cache instance of any config can get using `Nishchay::getCache` method. This method accepts one optional parameter.

```php
Nishchay::getCache();
```

Above will return instance of cache handler of default config. To get instance of specified config pass config name:

```php
Nishchay::getCache('configName');
```

All of the methods of cache instance are same for both cache server and it also works same.

###### Add/update item

To update or add an item into cache call `set` method. This method has following parameters:

| Name | Description |
|-----|-----|
| key | Key for the item |
| value | Item value |
| expiration | Expiration time in seconds for the item. Default is zero(No to expire). |

Example:
```php
Nishchay::getCache()->set('key', 'value', 1000);
```

###### Add/update multiple

To add multiple items at a time call `setMulti`. This method returns count of item which has been added or updated in cache. Each element of an items must be in `key`, `value` and `expiration` order. We can omit expiration value to make item not to expire.

```php
Nishchay::getCache()->setMulti([
    ['key1', 'value1', 1000] # Add key1 with value1 having expiration of 1000 seconds
    ['key2', 'value2'] # Add key2 with value2 having no expiration
]);
```

###### Add item

Method `add` item only if it does not already exists in cache:

```php
Nishchay::getCache()->add('key', 'value', 1000);
```

Parameter of this method are same as `set` method.

###### Get 

Using `get` method we can get an item from cache, this method returns `null` if there's no item for the passed key.

```php
$item = Nishchay::getCache()->get('key');
```

###### Get multiple

We can also get multiple items at item. If there's no item for the key it returns `null` for it:

```php
$items = Nishchay::getCache()->getMulti(['key1', 'key2']);
```


###### Remove item

To remove an item from cache, call `remove` method with key. It returns true, if item has been removed from cache.

```php
Nishchay::getCache()->remove('key')
```


###### Remove multiple items

To remove multiple items at a time pass list of keys which need to be removed in `removeMulti` method:

```php
Nishchay::getCache()->removeMulti(['key1', 'key2']);
```


###### Replace item

Replace method works same as `set` method but method does not adds item if it does not exists. It returns `false` if item does not exists in cache:

```php
Nishchay::getCache()->replace('key', 'value', 1000);
```