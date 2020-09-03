#### Life cycle

This page explains how request is processed by nishchay framework and what framework does at each step of an application. Before jumping into understanding of request life cycle, you must be familiar below few concepts.

###### Structure Validations and file processing

Nishchay framework iterates over each files and directory defined in an application and then validates it against defined structure definition. Once  file and directory are validated, view files are directly stored to its view collection class.

For the special class, its processed based on its class type. As an example of controller, nishchay first validates annotation defined on controller class then it processes each methods to extract routes. This processing differs for each special classes.

Once this is processed it gets stored collection classes.

**ABOVE STEP IS SKIPPED WHEN APPLICATION IS IN LIVE STAGE, TO UNDERSTAND MORE READ COLLECTION**

###### Collection

Nishchay stores each special classes and view files into their own collection classes. Here special classes are:

1. Controller
2. Entity
3. Handler
4. Event

Classes other than above are not stored in collection classes. `This collections are persisted to files for the current application version, if application stage is live`. When application is in live stage than on the first request all collection are persisted to files which makes application much faster in all following requests.

Special classes except entity are first processed and then stored to collection. So that on the following requests, these collections are fetched from persisted files and are directly used by the framework.

##### Lfe cycle

This section explains detailed life cycle of any request. You should be familiar with requests, response and events.

1. Load application configuration. At this point only `application.php` configuration file is loaded.
2. Prepare structure rule form structure definitions(SKIPPED if application stage is live and collections are persisted)
3. Iterate over each directory and files for(SKIPPED if application stage is live and collections are persisted):
    1. File and directory validation
    2. Register special classes and views to collection.
    3. If application stage is live, persist each collections to files for current application version.
4. If above step is skipped then collections are loaded from persisted files.
5. Starts stage.
6. Match requested url with registered routes.
7. Save details to current stage as per located route.
8. Checks
    1. Check if incoming request is allowed (Only applicable for the first stage). If application is running from console command then this check is skipped.
    2. If current stage is not the first stage then check ascent is allowed on located route.
    3. If located route is service then also check following
            1. Verify access token if service requires access token
            2. Validate fields demand.
9. Check maintenance modes.
    1. If requested route is in maintenance then start new stage with maintenance route. Processing jumps to step 5. This new stage is marked as maintenance stage so that when processing comes to this step, it will be skipped.
10. Create instance of located route's controller using Dependency injection.
11. Resolve controller property using Dependency injection.
12. Execute events which need to executed before route. This will execute events defined for the global, scope and on controller class and route method.
13. Check event response.
    1. If any of the event returns false then requests results in BadRequestException.
    2. If event response is not null and true then consider it as route response. In this case route method won't be called.
14. Calls route method using dependency injection. (Route method is not called if event returns false or event response is not null and true)
15. Execute events which need to called after route. This will execute events defined for the global, scope and on controller class and route method.
16. Handle response (This handles route response and/or event response if event returned response)
    1. Check if response is an instance of request forwarder. If response is request forwarder then start new stage with forward url.
    2. Redirect request is response is instance of request redirector.
    3. Handle response based on route response type.
17. This now sends response.