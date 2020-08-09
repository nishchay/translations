#### Logger settings

We can enable/disable logger using configuration `settings/configuration/logger.php`. We can also specify where log should stored and how log file should be rotated.

##### Enable/Disable

Using `enable` setting we can enable or disable logger within our application. Value should be boolean. When logger is disabled, trying log message results in an exception.

##### Log to file or database

We can store in log in file or database, this can done by setting `type`. Value of this setting can be `file` or `db`.


###### Log location file

Use `path` setting to set path where log messages should be logged. Here path must be directory. `path` setting is applicable for log type `file`.

###### Log location DB

Section `db` contains setting to log messages in db. This is applicable only when log type is `db`. See below settings

| Name | Description |
|-----|-----|
| connection | Database connection name. Keep it `null` to use default database connection. |
| table | Table name where log should be logged. |

Table strcture for log(MySQL database):

```sql
CREATE TABLE `RequestLog` (
  `requestLogId` bigint(20) NOT NULL AUTO_INCREMENT,
  `type` varchar(255) DEFAULT NULL,
  `message` text,
  `createdAt` datetime DEFAULT NULL
  PRIMARY KEY (`requestLogId`)
) DEFAULT CHARSET=latin1
```

##### Log rotations

Logging all logs into one files results in big file. Log file should be rotated date, week or monthly. Means there shoulld new log file after some duration. Nishchay allow to define this duration. Suppose if we set log to rotated to daily, then new log file will be created daily with file name as date in `Y-m-d` format.

Log rotation options.

| Name | Description |
|-----|-----|
| date | New log file will be created daily |
| week |New Log file will be created weekly and file name will date of first day of week. |
| biweek | To create log file every 15 days. File name will be for the first 15 day `Y-m-1` and another 15 days it will be `Y-m-16`. |
| month | To create log file once a month and in this case log file name will be first date for month. |

Log files are created in `Y-m-d` format only.