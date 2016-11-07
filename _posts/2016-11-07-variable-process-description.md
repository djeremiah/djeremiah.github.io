---
published: true
layout: post
author:
  name: Murph
---
# Including Process Variables in Process Descriptions in BPMS 6.3

When working with processes and tasks, it is sometimes advantageous to be able to set the process or task description. This can be used to convey valuable additional context to users, as well as differentiate between multiple instances of the same process or task.

By default, the process description is the same as the process name.
![Default process description matches process name]({{site.baseurl}}/images/default-process-description.png){:width="800px"}

We are able to alter this value by setting the "Process Instance Description" property in the process editor. This value can either be a static custom value, or it can include references to process variables. N.B., the variable expression is only evaluated once, when the process is first started. So any value you wish to add here should be passed in when starting the process. 
![Process Instance Description field in Process Editor]({{site.baseurl}}/images/process-instance-description-property.png){:width="800px"}

Note that the syntax here is `#{variable name}`.

The result is our custom decription appearing in the process instance list.
![Custom process description shows in instance list]({{site.baseurl}}/images/custom-process-description.png){:width="800px"}
