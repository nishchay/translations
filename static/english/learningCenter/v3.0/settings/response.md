#### Response settings

Response setting is for changing default response, setting view locator and enabling or disabling template engine for view.

###### Change default response

We can change default response type to XML or JSON and it will be applied to all routes where `@Response` is not defined. This can be done by changing value of `default`.

###### Define our own view locator

Nishchay find view by different methods. If we have defined structure within which Nishchay is unable to locate view, we can have our own locator so that we can implement our own view locator.

Keep value as `null` to use Nishchay view locator. If we are defining our own locator then it should be located in `Extension` namespace. Here if our class name is name is `Extension\View\Locator`, then value should be `View\Locator`. Locator class must extend `Nishchay\Http\View\AbstractLocator` class.

###### Template Engine

Using `templating` section we configure which templating engine to use. See below settings:

| Name | Description |
|-----|-----|
| engine | Keep this value `null` to use `TWIG` templating engine. To use some other template engine, define it in callback implementation. Place callback in 'Extension' namespace. For callback do not prefix it with `Extension` Example: Your callback located at ` Extension\View::templating` then value of this config should be `View::templating`. |
| caching | To enable or disable template caching. Use this setting to enabled or disable caching if we are using other templating engine |
| phpFunction |  Applicable for TWIG template engine only. Twig doesn't allow php function to be executed in twig template but setting below setting to true will allow it. |