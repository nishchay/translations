#### Request parameter
Class `Nishchay\Http\Request\Request` is used to fetch GET, POST request parameters. It can also be used to fetch request file.

##### GET parameter

Using `Request::get` we can fetch GET parameter. If get parameter with name does not exists it returns `false`. This method accepts only one argument which should be name of GET parameter.

```php
$name = Request::get('name');
```

Check if GET parameter `name` exists or not

```php
if (Request::get('name') === false) {
    # Do something
}
```

###### Fetch all GET parameters
Don't pass anything to `get` method and it will returns all GET parameter in an array. It returns empty array if no GET parameter exists.

##### POST parameter

Just like `get` method `post` works same but this method used to fetch POST parameter.

```php
$name = Request::post('name');
```

Check if POST parameter `name` exists or not

```php
if (Request::get('post') === false) {
    # Do something
}
```

###### Fetch all POST parameters
Don't pass anything to `post` method and it will returns all POST parameter in an array. It returns empty if no POST parameter exists in request body.

##### URL segment
To fetch placeholder of url  use `segment` method which accepts only one argument, name of segment. If segment with name does not exists it returns false.

```php
$userId = Request::segment('userId');
```

Check if POST parameter `name` exists or not

```php
if (Request::segment('userId') === false) {
    # Do something
}
```

###### Fetch all segment
Don't pass anything to `segment` method and it will returns all segments from url. It returns empty array if there are no placeholder segment in url.


##### Request file
Using `file` method we can get uploaded file. It returns instance of `Nishchay\Http\Request\RequestFile` which contains details of uploaded file. If there's no file with the name, it returns `false`.

`RequestFile` class have following methods using which we can get details of uploaded file

| Method | Description |
| ----- | ----- |
| getFileName | Returns name of uploaded file |
| getTempName | Returns path to temp file|
| getType | Returns name of uploaded file |
| getSize | Returns name of uploaded file |
| getError | Returns name of uploaded file |
| rename | To rename uploaded file |
| move | To move file from temp directory to actual upload directory |

If there are multiple file with the name, it returns list of `RequestFile` instance in an array.

###### Upload file

When server receives request with file, file is first uploaded to temp directory. We then have to move it to our upload directory. We can do that with the help of `move` method. This method accepts only one argument which should be location where file need to uploaded to.

```php
$file = Request::file('profilePic');
if ($file !== false) {
    $file->move('/Users/{serverUser}/images/profilePics');
}

```

**NOTE:** Do not pass file name with the location, `move` method will add file name(Name of file which returns in `getFileName` method). We only need to pass path to directory where file should be uploaded to.

###### Rename uploaded file

To rename uploaded use `rename` method. Pass only new name of file without any extension. This method reuse extension of uploaded file.

Its always good to rename file before upload it to upload directory to fix name conflicts.

```php
$file = Request::file('profilePic');

if ($file !== false) {
    $file->rename('newName')->move('/Users/{serverUser}/images/profilePics');
}
```

##### Fetch server value
Using `server` method we can server value. We can either pass server variable name or alias of it. Such alias is listed below:

| Method|Description|
| -----|-----|
| SOFTWARE | Server identification string |
| NAME | Name of the server host under which the current script is executing. If the script is running on a virtual host, this will be the value defined for that virtual host.  |
| IP | IP address of client |
| PORT | Port being used on the user's machine to communicate with the web server. |
| SERVER_IP | IP address of the server under which the current script is executing.  |
| SERVER_PORT | Port on the server machine being used by the web server for communication |
| SIGNATURE | String containing the server version and virtual host name which are added to server-generated pages, if enabled.  |
| ADMIN | The value given to the SERVER_ADMIN (for Apache) directive in the web server configuration file. If the script is running on a virtual host, this will be the value defined for that virtual host.  |
| HOST | Contents of the Host: header from the current request, if there is one.  |
| AGENT | Contents of the User-Agent: header from the current request, if there is one. This is a string denoting the user agent being which is accessing the page.  |
| ACCEPT | Contents of the Accept: header from the current request, if there is one.  |
| LANGUAGE | Contents of the Accept-Language: header from the current request, if there is one. |
| ENCODING | Contents of the Accept-Encoding: header from the current request, if there is one. |
| CONNECTION | Contents of the Connection: header from the current request, if there is one. Example: `Keep-Alive`. |
| QUERY | Query string, if any, via which the page was accessed.  |
| METHOD | Which HTTP request method was used to access the page |
| SCHEME | Request scheme like HTTP, HTTPS|
| URI | URI which was given in order to access this page |
| SCRIPT | Contains the current script's path. This is useful for pages which need to point to themselves. |
| PROTOCOL | Name and revision of the information protocol via which the page was requested |
| SELF | The filename of the currently executing script, relative to the document root. |

**NOTE** This method returns `FALSE` if server value with alias name or variable name not found.

##### Find if request is AJAX

Using `isAjax` method we can find if request is AJAX or not. This method returns `TRUE` if request is AJAX.

##### HTTP request methods
Below are some method which returns `TRUE` if based on request HTTP method.

| Method|Description|
| -----|-----|
| isPost | Returns `TRUE` if request HTTP method is POST|
| isGet | Returns `TRUE` if request HTTP method is GET|
| isPut | Returns `TRUE` if request HTTP method is PUT|
| isDelete | Returns `TRUE` if request HTTP method is DELETE|
| isPatch | Returns `TRUE` if request HTTP method is PATCH|
