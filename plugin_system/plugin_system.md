## Plugin System

A plugin can consist out of two modules, one of which is optional. The two modules are:
* Event Subscriber (required)
* Event Publisher (optional)

#### Event Subscriber
Event Subscriber listens for the incoming events comming from an event publisher. The event publisher either belongs to the same plugin
as event subscriber does, or it belongs to a forgein plugin. This allows for building plugins which depend on plugins.

An event subscriber can subscribe to many different event publishers. This module uses API provided by the event publisher it is 
subscribed to, to provide functionality to the app.

#### Event Publisher
Event Publisher must register itself to the Global Event Publisher Manager. An event publisher is responsible for listening for events and
dispatching the events in a wrapper API to subscribed event subscribers. An event publisher must not provide any other functinoality than
dispatching the events.

#### Global Event Publisher Manager
The GEPM controls event publishers, it can for example, tell a publisher to stop dispatching events which in turn shuts down
plugin's functionality – which might allow another plugin to fully take over the incoming events comming from enabled event publishers.

![](https://i.imgur.com/lJ2gH2B.png)
