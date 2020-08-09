#### Nishchay means 'decision'

`Nishchay` is an open structure PHP framework released under New BSD license in 2020 developed by `Bhavik Patel`.
Name of the framework chosen from 'To do anything, we only need to decide and start working on it'.
As the name suggests Nishchay has number of features which helps you to achieve your aim.

Open structure means it allows you to create any kind of application structure you want.
Once you get basic idea about Nishchay and open structure, you can learn more about open structure [here](/learningCenter/openStructure).

Before we start first demo of the framework please get basic understanding of few concepts.

*   Open Structure - Basics
*   Annotation
*   Stages of an application
*   Stages of request
*   Controller & Routes
*   Data Management

###### Open Structure

Most framework forces you to use structure which comes with it.
You are required to put every controller class in controller folder or sometimes controller inside module.
Your application can be either single level MVC (here single level means all the controller, model and views files placed in their dedicated folder) or hierarchical mvc.
Nishchay will follow what you have defined by using concept of open structure.

In other framework if you put view, model file inside controller directory it will allow you.
Although it get ignored it makes application ugly just because files should be placed to their type of directory.
Nishchay open structure processor prevents this from happening.
It would be good if your code and structure follow standard defined by you or provided by framework.
If someone tries to break, it should not be allowed.

Open structure allows you to create type of structure you want. Everything including controller, model, event, views, template etc. follows structure you define.
Nishchay structure processor supports only those things which are defined in structure definition.
This way your application gets pattern you define.

To know more about open structure like creating and managing, please click [here](/learningCenter/openStructure).

###### Annotation

Annotation is for declaring meta information about method & property of class and class itself or standalone function.
Although PHP does not support annotation, Nishchay has implementation to support it by allowing it to define in documentation comment.
Remember documentation annotations which are used to generate document does not have parameter while annotation which are used in Nishchay have it.
Any annotations which have parenthesis are counted as Nishchay usable annotation and are processed by Nishchay annotation parser.
But still there are annotations which do not have parameter and are used to define type of the class only.

Now and on we will talk about Nishchay usable annotation only and at any place annotation appears, consider it as Nishchay usable annotation.
Nishchay has predefined annotation and it supports those much only.
If you use annotation at any point Nishchay consider as it was declared by you.
If you define annotation which is not supported by the Nishchay, it throws error.
There are many annotations available in Nishchay and its uses & concept varies from feature to feature.

As already told annotation can have parameter. Every parameter have their key and value.
Because of key and value combination, parameter's order does not affect anything.
Means if annotation supports two parameter named 'foo' and 'bar', it doesn't matter if you place foo before or after bar parameter.
Every annotation have their supported key names and its depends on annotation and key itself.

Below is simple example of actual annotation which makes method to handler request.

```php
    /**
    * @Route(path="PathName")
    */
    public function methodName(){
        // Implementation
    }
```

Route annotation supports another parameter too, but for now to understand annotation, we will talk about one parameter only.
Here `path` is key and `PathName` is its value.

**NOTE:** Annotation definition is terminated by end line.

Below is example of invalid annotation.
This annotation definition is ignored by Nishchay annotation parser as definition is not terminated by end line.
```php
    /**
    * @Route(path="PathName")*/ 
    public function methodName(){
        // Implementation
    }
```
Points to keep in mind.

*   Double and single quotation does not make any differences.
*   If value of key is single word, quotation is optional.
*   NULL, TRUE and FALSE used without quotations are converted to their actual type. These three words are case insensitive means null is same NULL.

Nishchay annotations which does not support parameter are listed below.

1.  Controller
2.  Service
3.  Entity
4.  Doc
5.  Event
6.  Identity

###### Stages of an application

Stage of application is decided from the place where it is hosted. You develop your application on your local machine, this stage is known as local stage of an application. Once application developed it should be send to testing, this creates another stage of an application called test. Once application passes all tests, it become ready to go live. This stage is known as live stage of an application.

Nishchay does not detects which stage of an application is active, it should be declared by you by changing stage configuration value in application setting.

Learn more application setting [here](/learningCenter/settings/application).

###### Stages of request

Nishchay allows request forwarding within server, so whenever requests comes it starts with stage 1. This number keep increasing by 1 on each request forward.

Each stages of an request have their own request detail like located route, its method, controller, context and etc.

Learn more about request [here](/learningCenter/request/requestHandling).

###### Controller & Routes

Class which handles request are known as controller. Any class can be controller. To make class a controller you only need to annotate that class with 'controller' annotation. Controller consists of one or more routes which actually handles request. Controller makes method to support route and any other related annotation so the method can handle request and they can have required features.

Know more about [controller](/learningCenter/request/controller) & [routes](/learningCenter/request/requestHandling).

###### Data Management

Nishchay provides support for managing data within your application that's database. Two ways you can deal with data, Using [Entity Manager](/learningCenter/data/entity) or [Query Builder](/learningCenter/data/queryBuilder). Apart form this you can also change database table structure using [Database Manager](/learningCenter/data/databaseManager).