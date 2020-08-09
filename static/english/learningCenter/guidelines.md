#### Guidelines

Here are some security and coding guidelines which you should follow to make your application secure, faster and easy to maintain. This page does not covers everything but it lists basics which you should follow.

##### Coding standard

Your code must follow some set of coding standards like intending, control structure variable placements. Because there can be many developer for an application, using coding standards application follows same pattern so that each developer of an application can easily understand each code.

We recommend following [PSR](https://www.php-fig.org/psr/) coding standards. Take a time and read each coding standards and then start following these coding standards in your application.

###### Security consideration

To keep your application secure, you should have follow below things:

1. Always use CSRF for the form in view.

2. Always use user input validation either using `Validation` or `Form` class.

3. Don't directly print user input in view. If you are using TWIG view then you don't have to worry about but use `{varName|raw}` only if it does not contain HTML, Javascript value.

4. Don't execute directly raw query, even if you use then always bind user input. First thing is to try building query using `EntityQuery` and `Query` builder.

5. We recommend using `Entity` class, as it comes validation, property value type casting, events and more.

6. Use `encrypt = true` for sensitive data in `Entity`. These sensitive can include personal, financial information.

7. Don't put sensitive information in `Cookie` session and pass data to view only if its required.

8. We recommend implementing service with `token = true` for secure services.

9. Uploaded files must be placed outside of your application. These files must be validated first either using `Validation` or `Form`
class. Application must not allow any kind of executable file to be uploaded.

There are many other security consideration which you should follow but we have listed only those which are basic. Such security consideration can be done through remote server configurations, database configurations.


###### Application maintenance

1. Log each error which occurs within your application.

2. We recommend using translations for each line and word. Even if your application to be meant for only one language, translations can remove repetition for same line or word.

3. We also recommend using Git, Bitbucket for version control system. This allow many developer to work on same project at same. It also helpful for rolling back application version to previous version, tracking changes in your application.

4. To make your application or say deploy your application to remote server, use automated deployment tool like github webhook, bitbucket pipeline, jenkins and there many other automated deployment available on internet.

