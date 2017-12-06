---
title: Dynamic receive timeouts for Camel-CXF
published: true
layout: post
author:
  name: Murph
---
Setting CXF Client Timeouts Dynamically in Camel
======================

I have published a new gitlab repo that demonstrates how to dynamically set the ReceiveTimeout on a camel-cxf component at runtime.
[camel-boot-cxf-dynamic-timeout](https://gitlab.com/murph83/camel-boot-cxf-dynamic-timeout)

It contains two projects:

timer-service
------------------------
This project implements a *very* simple SOAP service, as defined by [TimerService.java](https://gitlab.com/murph83/camel-boot-cxf-dynamic-timeout/blob/master/timer-service/src/main/java/rh/demo/TimerService.java). It will wait for the input number of milliseconds before returning a success message. This allows us to play around with expected timeouts.


cxf-dynamic-client
-----------------------------
This project demonstrates the use of CXF Request Context to override the default receive timeout on a per-request basis.

In particular, [CxfUtil.java](https://gitlab.com/murph83/camel-boot-cxf-dynamic-timeout/blob/master/cxf-dynamic-client/src/main/java/rh/demo/CxfUtil.java) has a method `setTimeout` that performs the work.
```java
public void setTimeout(Exchange exchange, int timeout, int timeoutOffset) {
        // Create ClientPolicy
        HTTPClientPolicy clientPolicy = new HTTPClientPolicy();
        clientPolicy.setReceiveTimeout(timeout + timeoutOffset);

        // Populate RequestContext with the Policy and set it on the exchange
        Map<String, Object> requestContext = new HashMap<>();
        requestContext.put(HTTPClientPolicy.class.getName(), clientPolicy);
        exchange.setProperty(Client.REQUEST_CONTEXT, requestContext);
    }
```
Note that this method has an extra parameter `timeoutOffset`. This allows us to play around with whether the timeout should be greater than the requested timeout, or less than. 
```xml
<!-- First request should succeed -->
<to uri="bean:cxfUtil?method=setTimeout(*, ${header[requested.timeout]}, 500)"/>

<!-- Second request should time out -->
<to uri="bean:cxfUtil?method=setTimeout(*, ${header[requested.timeout]}, -500)"/>
```

Another interesting thing to note is that the camel-cxf component will try read the RequestContext from both the [message headers](https://github.com/apache/camel/blob/master/components/camel-cxf/src/main/java/org/apache/camel/component/cxf/DefaultCxfBinding.java#L494) and the [exchange properties](https://github.com/apache/camel/blob/master/components/camel-cxf/src/main/java/org/apache/camel/component/cxf/DefaultCxfBinding.java#L506). Since the exchange properties are
checked second, they'll win. This means that if you make multiple cxf calls in the same exchange (as in our example route), the RequestContext from the first call will propagate to the second call. This is excellent if you need properties to persist between calls, but not if you want the calls to be fully independent. That is why the sample code sets the new RequestContext to the exchange, rather than using a message header.

Testing
-------
In the root directory, there is a script `start.sh` that will start the timer-service and cxf-dynamic-client spring boot applications.

Then, you can invoke `test.sh` to start the test. The flow looks like this:

![test flow]({{site.baseurl}}/images/testflow.png)

The test script expects to see the timeout exception, and will report success only if it is present. You can review the full output of the services in the `*.log` files created in the root directory when the services start.

Once the test is complete, you may run `stop.sh` to terminate the services.

