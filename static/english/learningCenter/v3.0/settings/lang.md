#### Language/Translations settings

Language settings contains configurations like setting main or default locale. Here we can also specify language directory where all translations are located. Language setting is located at `settings/configuration/lang.php`.

##### Set locale

Inside `locale` section we can define `main` or `default` locale. `main` is main locale for the application.  If translation does not found in main locale then `default` locale. IF translation does not found in both of these locale then exception is thrown.

| Name | Description |
|-----|-----|
| main | Main locale from which translation will be fetched. |
| default | If the translation does not found in main locale then translation fetched from below mentioned locale. If default locale is not valid or does not contains translation then exception is thrown. |

##### Translations type

We can store translations into files or db and this can be specified in `type` setting. Value of `type` can be `db` or `file`. Based on this setting its related setting also need to be configured.

###### File

When we store translations into files, below settings need to be configured in `file` section. We can store translations into JSON or PHP file.

| Name | Description |
|-----|-----|
| type | Type of file JSON or PHP |
| path | Path where translations are stored. |


###### Database

For translations which are stored in database below settings need to be configured in `db` section.

| Name | Description |
|-----|-----|
| connection | Database connection name. Keep it `null` to use default database connection |
| localeAs | Set `column`, if we are storing translations in one table otherwise it should be `table` which allows us storing translation into its own table for locale. Locale table  name follows `AppLang{locale}` format. Example: For locale english, table will be AppLangEnglish |
| table | Applicable only when `localeAs` is `column`. Specify table name where translations are stored. |
| cacheExpiry | Cache expiry for translations loaded form database. Set it to `false` to disable caching |

Its good to have cache enabled for translations stored in database because it will prevent frequent query to db.

Whenever translations are updated and we have caching enabled, we need to clear cache so that application gets updated translations. We can get cache key by calling below method.

```php
$cachKey = Nishchay::getLang()
                ->getLoader()
                ->getCacheKey('english', 'english');
```

We might need to create MD5 of cacheKey, if `hash = true` to set in cache config. Here first parameter of `getCacheKey` is `locale` and second is `namespace`.

**NOTE:** Root namesapce should be same as locale name.

