#### Open Structure

Open structure allows us to create type of application structure we want. Everything including controller, model, event, views, template etc. follows structure we define. Nishchay structure processor supports only things which are defined in structure definition. This way our application gets pattern we define.

Structure definition located at `{APP_PATH}/settings/configuration/structure.xml`.

##### Structure definition

*   Structure definition is xml file which must contain only one root element.
*   Root element name should not be from _extension, bootstrap, public, logs, tests, vendor, settings, resources, persisted._
*   Application code will be contained in directory whose name matched with root element name.
*   Each element in definition is known as directory name.
*   Elements are of two types:


| Special| Other |
| -----|-----|
| Controller, Entities, views, events, handlers are special elements. Special element name is case insensitive. Except views, all other special element names are converted to First character capital and remaining lower. If defined <small>controllers</small>, it will be converted to <small>Controllers</small>. | Elements which have name other than special element are known as other type of elements. Here element name is case sensitive. We can put any type of class in Other elements. Structure processor will ignore type check in this directory.|


*   There should not be special element inside special element.
*   Empty elements denotes it should contain files within it. 

###### Element attributes

Elements can have attributes which further extends it capability. Attributes are ignored if they are not from attributes which are mentioned below and/or if its value is zero.

| Name| Description |
| -----|-----|
| require | Makes directory mandatory. |
| nest | Allows directory to nest inside. Setting its value to '1', it will allows nesting of 5.3. Means we can create at most 5 directory inside it and each of these directory can contain at most three directory. You can learn more about in `nest attribute` section. |
| continue | This will allow us create same type of directory on same level. continue attribute is not applicable on special element. Even if we define it on special element, it will be ignored. |
| root | Allows to create file on root level. By default, files are allowed in empty element only. We might need this attribute when we are continuing or nesting directory and want to allow files to be created on root level. |
| type | Allows us to define class type of classes which are other than special type. Please read more about _Class type_ in its section below. |

##### Basic Structure

Below is the structure definition which comes with installation.
```xml
<Application>
    <Controllers nest='1' require='1'>
        <!-- Routes which should be accessed directly. -->
        <Direct/>
        <!-- Routes which requires login into application. -->
        <Secure/>
        <!-- All routes which are services. -->
        <Service/>
    </Controllers>
    <Entities nest='1'/>
    <Models type='model' nest='1'/>
    <views require='1'/>
    <Events/>
    <!-- Exception handlers -->
    <Handler/>
    <!-- Form classes -->
    <Forms type='form'/>
    <!-- Various feature implementations -->
    <Features type='feature'/>
</Application>
```
1.  Here whole application code will be contained in `Application` directory.
2.  There are three type controllers in it. Direct which will contain routes which will accessed directory while Secure will contain routes which requires login into account. Service contains all services within application. These directories are created so that secure, non secure and services routes are placed in their appropriate directory.
3.  This structures also forces Controllers, views directory to be exists in an application.

##### Continue attribute

Continue attribute allows same type of element to be created on same level.

*   For this element, directory name can be any.
*   value = 0, attribute will be ignored.
*   Settings its value to '1', will allows at most 5 on same level.
*   It supports any number and it will allow that number of directory on same level.

**Example:**
```xml
<Application>
    <Feature continue='1'/>
</Application>
```
As Feature element has `continue` attribute set to 1, structure processor will allow us to create at most 5 directory on same level. Because it has  `continue` attribute, we can create directory with any name. If it does not had continue attribute then director name should be element name.

Using above we can create below directory as per structure definition.
```xml
<Application>
    <Account>
    <Message>
    <Groups>
    <Pages>
    <Help>
</Application>
```
##### Nest attribute

Nest allows directory to be nest inside. Setting its value to any number will make that directory to behave like it has continue attribute.

We can also define nest rule. Nest rule are consist of number of depth limits. Depth limit is defined within opening and closing curly braces. General form of depth limit is shown below.
```
{DEPTH_PATH,LIMIT}
```
Depth path starts with root point, root point is named as `R`. Each depth is concat by dot. asterisk is wildcard depth point, which stands for all. Here are some depth limits, {R,5}, {R,5}{R.*,3}, {R,5}{R.0,3}.

###### Root directory: {R,5}

Allows to create at most 5 directory at root. As there are no other rule along with it, each directory should contain file only. Suppose we have Feature element with above nest rule, then we can define at most 5 directory inside Feature directory. Such examples are _Feature/Account_, _Feature/Message_. But we can not create any directory inside _Feature/Account_, creating inside it will throws `InvalidStructureException`.

If we want to allow each of these directory to contain another directory within it. We should use R.*, please see below example.

###### Wildcard depth point: {R,5}{R.*,3}

Above rule extends first example by allowing each root directory to contain another directory within it. Each directory _Feature/Account_, _Feature/Message_ can have at most 3 directory within it. For _Feature/Account_ we can create _Feature/Account/Profile_, _Feature/Account/Setting_, _Feature/Account/Privacy_

###### Specific directory: {R,5}{R.0,3}

Depth point starts form zero. If we want to allow only second directory to have child, we can use number instead of wildcard. **{R,5}{R.0,3}** will allow first directory to have at most 3 child.

##### Class Type

We can create custom type of class by defining type attribute on other type of element(Element other than special element are known as other type). Once we define type attribute on other element it will force us to define `@ClassType` to be define on class.

`@ClassType` supports only one parameter called `type` whose value must be same as type attribute defined on other element.
```php
/**
* @ClassType(type='feature')
*/
class AccountFeature {
```
