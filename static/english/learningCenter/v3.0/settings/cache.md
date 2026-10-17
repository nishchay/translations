#### Cache settings

This setting contains cache related configuration which is located in `settings/configuration/cache.php`. We can define list cache configuration to be used within application. This setting also cache for each database connection. 

##### List
Inside `config` we can define list of cache configuration in an array where key should cache config name. This setting should have following configuration.

| Name | Description |
|-----|-----|
| type | Should be `memcached` or `redis` |
| server | Read `memcached` and `redis` section of its server configuration |
| offline | To disable this cache configuration |
| hash | Whether to hash passed key |

###### Configuration: memcached

We can define one or more servers of memcahed with following configuration. 

**NOTE** Configuration order need to be same as mentioned below.

| Name | Description |
|-----|-----|
| host | Host name or ip of memcached server |
| port | Port number where memcached is running. Usually, this is 11211 |
| weight | Weight of the server relative to the total weight of all the servers in the pool. This controls the probability of the server being selected for operations. This is used only with consistent distribution option and usually corresponds to the amount of memory available to memcache on that server.  |

###### Configuration: redis

Unlike `memcached`, in `redis` we can define only one server. In redis server configuration order does not matter. We can configuration except `host`.

| Name | Description |
|-----|-----|
| host | Host name or ip of redis server |
| port | Port number where memcached is running. If this setting omitted then default to 6379 |
| timeout | Timeout for connection, default to 1 second |
| retryInterval | After each connnection failure how miliseconds to wait for. Default is 100 |
| readInterval | Read interval in second. Default is 1 second |


##### Disable/Enable cache

We can disable all cache by setting `enable = false`. To disable individual cache configuraiton set `offline = false` of that configuration.

When Caching is disabled then, while adding data to cache will return `false` and when we fetch result always be `null`.

##### Database cache

Database related cache configuration located in `database` section where we only have mention cache configuration name as defined in `config`.

Inside `connection` we can define which cache to use for which database. If we do not specify database connection here then no cache will be used for that database connection. `connection` shoould list of caching for database connection where key should be database connection and its value should be cache configuration name.


###### Default database cache

We can define default cache for all database connnection which listed in `connection`. Here setting `null` will use default cache as defined for main `config`.

If we specify `null` for any database connection then cache defined in `default` will be used.