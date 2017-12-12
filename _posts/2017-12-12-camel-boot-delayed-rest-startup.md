---
title: Delayed Startup for REST DSL Routes
published: true
layout: post
author:
  name: Murph
---
Delaying Startup for REST DSL Routes in Camel
======================
When starting up a Camel application, you may wish to delay consumption of messages until some initialization is complete. 
This is usually seen when your routes are attached to messaging endpoints, but can also apply to REST endpoints. 
The system can send a 404 until the initialization is complete and the route is started.

Setting this up is simple:
1. First, create the rest endpoint in the REST DSL. Use an explicit `<route>` child to allow you to set a route id and turn off auto-startup.
    ```xml
    <rest path="/api">
        <get uri="/resource" >
            <route id="get-api-resource" autoStartup="false">
                <to uri="direct:get-api-resource"/>
            </route>
        </post>
    </rest> 
    ```

2. Next, we need to create the initialization logic that will prepare the application. The final step of this logic will be to start the REST route.
There are a few ways to accomplish this. I'll cover two options.
    * Using a run-once timer to kick off an initialization route
    * Using the [EventNotifier](http://camel.apache.org/eventnotifier-to-log-details-about-all-sent-exchanges.html) api

    Timer Route:
    ```xml
    <route id="startup-route">
        <from id="startup-route" uri="timer:startup?repeatCount=1"/>
        <log message="Initializing application"/>
        <!-- perform initialization logic here -->
        <to uri="controlbus:route?routeId=get-api-resource&amp;action=start"/>
    </route>
    ```

    EventNotifier:
    ```java
    @Component
    public class StartupLogic extends EventNotifierSupport{

        public void notify(EventObject event) throws Exception {
            CamelContext camelCtx = ((CamelContextStartedEvent)event).getContext();
            // perform initialization logic here
            camelCtx.startRoute("get-api-resource");
        }

        public boolean isEnabled(EventObject event) {
            return event instanceof CamelContextStartedEvent;
        }
    }
    ```
