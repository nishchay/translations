#### Application Settings

Application settings divided into three section called application, config and session.

Application section contains information regarding our application like name of an application and its author. `stage` is application development stage, `local` means in development, `test` means its Testing phase. Once application passes `local` and `test` phase it gets ready for go `live`.  Stage `live` differs from other two stage as in this stage error won't be displayed.

Once we move application to live stage. Nishchay pesists all routes to persisted directory so that it can be used directly. This is to prevent Nishchay processor to go thru whole code on each request. This improves application processing time. Nishchay processor persist processed routes, controller in live stage only.

Nishchay processor creates directory with version name, persists routes, controllers into it. It is required to change to version name once you change controller or route from its previous version.

See below application settings:

| Name|Description |
| -----|-----|
| name | Application name |
| author | Application author name |
| stage |Current stage of an application. Which can be any of `local`, `test` or `live` |
| version | Application current version number |


##### Config section

| Name|Description |
| -----|-----|
| fileTypes| List of supported file extension separated by pipeline(|). If we remove php from this list, our application won't support php files too. So please do not remove php from this list.|
| landingRoute | Entry point of our application. This is the route from our application starts |
| resourcePath | Where all our static files are placed like JS, CSS and images. This should be path from our project |

##### Session

From this setting we can configure where session data should be persisted.

| Name|Description |
| -----|-----|
| storage| storage type like file, db or cache. Default value is file. This can be changed to db or cache. Changing this requires its related settings to be modified.|
| storagePath | Path to directory where session data need to be stored, We recommend this to be outside of application. |
| db | Required when storage is db. See DB configuration. |
| cache | Applicable only when storage type is `cache`. See cache configuration |


###### DB configuration

When Session storage is `db` then below config should be specified in `db` section.

| Name | Description |
| -----|-----|
| connection | Connection name, keeping it NULL uses default connection as provided in database setting. |
| table | Table name where session data will be stored, keep it NULL will consider `Session` as table name. |

Table structure for `Session`:

```sql
CREATE TABLE `Session` (
  `sessionId` VARCHAR(250) NOT NULL,
  `data` TEXT DEFAULT NULL,
  `accessAt` BIGINT DEFAULT NULL
) DEFAULT CHARSET=latin1
```

###### Cache configuration

When session storage is `cache` configure bellow settings in `cache` section.

| Name | Description |
| -----|-----|
| name |  Cache config name, it should be null if want to use default cache config as defined in cache.php config file. |
| expiry | Cache expiry time in seconds, this should be greater than 300 seconds. |