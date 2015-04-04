.. _simple_vm_application_label:

Demo Application: Simple VM
---------------------------

.. todo:: Move css that fixes text width into css stylesheet

.. raw:: html

    <style> p {max-width:750px; width:100%; font-size:18px; color:rgb(0,50,100); text-align:left;}  
            strong {font-size:18px;}
            li {font-size:18px;}
            div {max-width:750px; width:100%;}
            h1 {font-size:26px; color: black; padding-top: 125px; padding-bottom: 125px;}
            h2 {font-size:24px; color: black; padding-top: 125px; padding-bottom: 125px;}
            h3 {font-size:22px; color: black; padding-top: 125px; padding-bottom: 125px;}
                
    </style>



What is Simple VM?
~~~~~~~~~~~~~~~~~~

This tutorial uses a demo application Simple VM to demonstrate how create you own Murano Application.
This section will walk you through its source code line-by-line and then explain how to upload, deploy and test it.

Simple VM is a very basic application: it creates instance and provides an action that can restart it trough Murano Dashboard.

The source code of Simple VM is available in github:   
https://github.com/gokrokvertskhov/murano-trainings/tree/master/Day_1/io.murano.apps.test.SimpleVM


Simple VM Code Explained
~~~~~~~~~~~~~~~~~~~~~~~~

What is inside?
_______________

The package structure of Simple VM is::

  ..
   |_  manifest.yaml
   |
   |_  Classes
   |   |_  SimpleVM.yaml
   |
   |_  Resources
   |   |_  Restart.template
   |   |_  scripts
   |       |_restart.sh
   |
   |_  UI
   |   |_  ui.yaml
   |
   |_  logo.png
   
   
The structure of the typical Murano Application is described here:
.. todo:: add link

The interesting part of Simple VM is that it does not install any software on the instance it creates.
That's why there is no any .sh script that would install software and consequently there is no template for install script in Resouces folder.
On the other hand, Simple VM provides a restart action, which is available at Murano Dashboard.
The logic of Restart action is placed in Restart.template and restart.sh.  

manifest.yaml
_____________

**manifest.yaml**::

   Format: 1.0
   Type: Application
   FullName: io.murano.apps.test.SimpleVM
   Name: Test SimpleVM
   Description: |  
    This is a simple test app with a single VM.
   Author: 'Mirantis, Inc'
   Tags: [test]
   Classes:
    io.murano.apps.test.SimpleVM: SimpleVM.yaml

Source code may be downloaded from here: 
https://github.com/gokrokvertskhov/murano-trainings/blob/master/Day_1/io.murano.apps.test.SimpleVM/manifest.yaml

This is a manifest file. It contains general information about the application.
It is very similar to the manifest of a jar archive.
It has a fixed format based on yaml (this is not MuranoPL).

Let's inspect manifest.yaml line-by-line:

::

   Format: 1.0

Specifies the version of the format for manifest.yaml. Currently 1.0 is the only available version.


::

   Type: Application
   
Specifies the type of the package. There are only two possible values: ``Application`` and ``Library``.
The value ``Application`` is used for all Murano Applictions. The value ``Library`` is a special case. 
It is used for a package that contains classes common to multiple applications.
The example of a library, is ``Core Library``. It contains core Murano API (http://murano.readthedocs.org/en/latest/murano_pl/core_classes.html) 
You can see Core Library on the Packages Panel:

.. image:: images/packages.png

Note, that both values: ``Application`` and ``Library`` start with capital letter. This is the naming convention for all pre-defined values in Murano code.

::
 
   FullName: io.murano.apps.test.SimpleVM
   
This is a globally unique name of a Murano application. 
To ensure the global uniqueness, we follow the same naming convention as the naming convention of Java packages and classes.
The "io.murano.apps.test." is a "package" part of the name.
"SimpleVM" is a "class" part of the name.


.. note:: **Java Language Specification states**: You form a unique package name by first having (or belonging to an organization that has) an Internet domain name, such as oracle.com. You then reverse this name, component by component, to obtain, in this example, com.oracle, and use this as a prefix for your package names, using a convention developed within your organization to further administer package names. Such a convention might specify that certain package name components be division, department, project, machine, or login names. The first component of a unique package name is always written in all-lowercase ASCII letters and should be one of the top level domain names, such as com, edu, gov, mil, net, or org, or one of the English two-letter codes identifying countries as specified in ISO Standard 3166. The name of a package is not meant to imply where the package is stored on the Internet. The suggested convention for generating unique package names is merely a way to piggyback a package naming convention on top of an existing, widely known unique name registry instead of having to create a separate registry for package names. [...] Names of class types should be descriptive nouns or noun phrases, not overly long, in mixed case with the first letter of each word capitalized.

::

   Name: Test SimpleVM
   
Display name of an application. You will be able to reset a display name when you upload SimpleVM package to Murano.


::

   Description: |  
    This is a simple test app with a single VM.
   Author: 'Mirantis, Inc'
Self-descriptive piece of code, but lets's take a note on the syntax.

* ``|`` - vertical line symbol comes from YAML syntax. It means that all the leading indents should be preserved. This is very useful for long, multy-line descriptions.
* double white space start the line ``This is a simple test...``  It will be preserved when description is rendered on the Murano Dashboard.
.. todo:: test this 

.. warning:: Avoid TAB symbols inside yaml files. If yaml contains a TAB symbol, if will not be parsed correctly. The error message may be criptic or misleading

::

   Tags: [test] 
   
This is array of tags. You can search an application by its tag.
You may want to specify several tags for one application ``Tags: [test, Web]``
YAML offers two equivalent syntaxes to write an array

::

   Tags: [test, Web]

is the same as

::

   Tags:
     - test
     - Web
     

::

   Classes:
    io.murano.apps.test.SimpleVM: SimpleVM.yaml
    
This section is a mapping between all classes present in SimpleVM application and the file names
where these classes defined in. This is one-to-one relatioonship: i.e. one and only one class per file.
The line ``io.murano.apps.test.SimpleVM: SimpleVM.yaml`` says that the class ``io.murano.apps.test.SimpleVM`` 
is defined in the file ``SimpleVM.yaml``
If you had another class, say, ``io.murano.apps.test.ComplexAction``defined in file ``ComplexAction.yaml``
your Classes section would be

::

   Classes:
    io.murano.apps.test.SimpleVM: SimpleVM.yaml
    io.murano.apps.test.ComplexAction: ComplexAction.yaml
    
From syntax point of view, this is the example of YAML Associative array aka dictionary. 
Similar to array, it has one-line (== inline) syntax

::

   Classes: {io.murano.apps.test.SimpleVM: SimpleVM.yaml, io.murano.apps.test.ComplexAction: ComplexAction.yaml}
    
    
If I may have multiple classes within my Application, what is the entry point?
How Murano knows, which one is the "main" class?
The answer is that the "main" class is named exacly as the package FullName. In SimpleVM it is ``io.murano.apps.test.SimpleVM``

The main class must extend Murano Core class ``io.murano.Application``
and it must implement two method ``initialize`` and ``deploy`` 


See more details about Murano Classes in the section:  :ref:`simple_vm_yaml_label` 
 
 
.. _simple_vm_yaml_label:

Classes/SimpleVM.yaml
_____________________

**Classes/SimpleVM.yaml**::

   Namespaces:
     =: io.murano.apps.test
     std: io.murano
     res: io.murano.resources
     sys: io.murano.system
     apps: io.murano.apps
   
   #This is a name of a Simple VM Class
   #
   Name: SimpleVM
   
   # This is a parent class fo SimpleVM.
   # Applicatoin class has two methods:
   #  initialize
   #  deploy
   
   Extends: std:Application
   
   Properties: 
     name:
       Contract: $.string().notNull()  
       
     instance:
       Contract: $.class(res:Instance).notNull()
            
     host:
       Contract: $.string()
       Usage: Out
   
     user:
       Contract: $.string()
       Usage: Out
   # Workflow section is deprecated
   # Use Methods instead.
   #Workflow:
   Methods:
     initialize: 
       Body: 
         - $.environment: $.find(std:Environment).require()
   # This is a deploy method for our new app
   # This method will be called from Environment class method deploy.
     deploy:
       Body:
         # Attributes are runtime properties which are not visible to the user.
         # Use attributes to store some internal data between deployments.
         # Attributes data is persistent. It is stored as a part of Object Model.
         # $.getAttr(name, default_value) - gets attribute from the Object model
         # $.setAttr(name, value) - store attribute value in the Object Model
         # Only basic types are supported (boolean, int, string)
         - If: not $.getAttr(deployed, false)
           Then:
             # This is a log method to send status log message back to UI
             # This is what users will see in the UI during deployment time
   
             - $.environment.reporter.report($this, 'Creating a VM ')
   
             # Security groups object is a list of dicts [{}]
             # each dict element in this list should have following key:value entries:
             #  FromPort: value - lower boundary of the port range
             #  ToPort: value - upper boundary of the port range
             #  IpProtocol: <tcp|udp> - transport protocol type TP or UDP
             #  External: <true|false> - if true it opens for CIDR:0.0.0.0/0
             #                           if false it opens port only for SecGroup members
             - $securityGroupIngress:
                 - ToPort: 22
                   FromPort: 22
                   IpProtocol: tcp
                   External: True true is more correct
             - $.environment.securityGroupManager.addGroupIngress($securityGroupIngress)
             # Now lets ask our instance to deploy itself.
             # Inside deploy method there are additional steps which configure networks for the environment
             - $.instance.deploy() 
             - $.environment.reporter.report($this, 'The new VM is created')
             # Save host information to local variables
             - $.host: $.instance.ipAddresses[0]
             - $.user: 'root'
             # Format log message based on variables values
             - $msg: format('{0}@{1}', $.user, $.host)
             - $.environment.reporter.report($this, 'The VM is available ' + $msg)
             - $.setAttr(deployed, true) True is string, its ia a bug, bool is true
   
     restartVM:
       Usage: Action
       Body:
         - $.environment.reporter.report($this, 'Restarting the VM')
         # Create new Resources class to have an access to the package resources
         # Package resources are stored in Resource folder
         - $resources: new(sys:Resources)
         # Load yaml file with execution plan. Bind execution plan parameters with actual values.
         - $command: $resources.yaml('Restart.template').bind(dict(
             time => 'now'
             ))
         # Send execution plan to the agent on the instance
         - $.instance.agent.call($command, $resources)
         - $.environment.reporter.report($this, 'Restart command was sent to VM')
   

Source code may be downloaded from here: 
https://github.com/gokrokvertskhov/murano-trainings/blob/master/Day_1/io.murano.apps.test.SimpleVM/Classes/SimpleVM.yaml

Let's inspect it line-by-line:

::

   Namespaces:
     =: io.murano.apps.test
     std: io.murano
     res: io.murano.resources
     sys: io.murano.system
     apps: io.murano.apps

Namespaces in Murano are like namespaces in XML.
Default namespace ``=:`` 

::
   
   Name: SimpleVM 
   
The name of the current class. Default namespace is ommitted.

::
      
   Extends: std:Application  
   
This clause specifies, that class ``io.murano.apps.test.SimpleVM`` inherits from  ``io.murano.Application`` 
MuranoPL allows multiple inheritance. In case if you class inhrerits from several classes, pass them as an array.
Please note, that this code will cause parsing error ``Extends: [std:Application, base:MyBasicVM]`` bacause in contains ``:``
Either pass the  fully-qualified class names as ``Extends: [io.murano.Application, io.murano.mybase.MyBasicVM]`` 
or use ``-`` array syntax::
   
   Extends: 
     - std:Application
     - base:MyBasicVM  
   
Then goes the section that describes class's properties.
All properties are "public" - you can reach and read them outside the instance of SimpleVM class.
There are various types of properties. In this example we have only ``In`` and ``Out`` types of properties.

Let's take a look at the ``In`` properties: ``name`` and ``instance``. 
We do not specify their type explicitly, so they gat default type which is ``In``

::
   
   Properties: 
     name:
       Contract: $.string().notNull() 
       
The sign ``$`` denotes current context. In this case it is the ``name`` property of current instance of SimpleVM class   ??? == $this.name.string().notNull()
The Contract clause enforces that the ``name`` is a non-empty string.
This is how it works: 
When the value of ``name`` propety will be passed to the SimpleVM instance, 
the function ``string()`` will be called on this value. Every MuanoPL function either returns value, or throws error.
string() function tryes to convert the value to string. If conversion is successfull, it returns the string value, 
if not, it throws error.
If string() returns a value, then notNull() funtion will be called on the string value.
notNull() function throws error, if value is null, otherwise returns the value itself.

``instance`` property is a non-null objects, that represents Virtual VM (==aka instance you see in Murano dashboard)

::   

     instance:
       Contract: $.class(res:Instance).notNull()

This contract ensures that instance parameter  is an instance of class Instance and that it is not null.       
The function call ``class(res:Instance)`` attemps to convert ``instance`` to the object of type ``io.murano.resource.Instance``
The MuranoPL object (i.e the instance of MuranoPL class) is a dictionary  of key-values pairs. Every object has a unique (string) id. 
Object has a two representaions:
          -- as a dictionary somewhere in the object model
          -- as an id
 
Object is represented as a dictionary only once  in the object model. The object that includes it is called OWNER.
Anywhere else in the object model the object is referenced by its ID.
Thus we have two independent object hierarchies: Inherinace and Owner, plus a graph of references between objects.
Class has constructor new that creates instances of this class


::
     
     host:
       Contract: $.string()
       Usage: Out
   
     user:
       Contract: $.string()
       Usage: Out

The properties ``host`` and  ``user`` are ``Out`` properties. The methods of this class will set value to them. 
Let's take a look at the first of the methods, ``initialize``

::

   Methods:
     initialize:
       Body: 
         - $.environment: $.find(std:Environment).require() 


This is a "constructor" of SimpleVM class. The lifecycleof a SimpleVM object is:

* load object model
* set properties
* validate properties
* execute initialize

Use this method to initialize private properties of SimpleVM, that are not visible outside the particular instance of SimpleVM class.
``$.environment:`` is a declaration of private property
The values of this property is returned by function .find(). It searches the owner object which type is Environment up in the OWNER hierarchy 
Here ``$`` means the environment peoperty of current instance.
It is the same as ``$this.environment`` 

Function ``require()`` verifies if ``find()`` function returns any value. If no value is returned, it throws ``error``. 


::
 
     deploy: 
       Body:
       
The ``deploy`` method of your 'main' class in an entry point to your application.
It is inherited from ``murano.io.Application``.
The ``deploy`` method has no parameters, but MuranoPL methods may take parameters
.. todo:: add reference to the example of a method thattaked parameters

::

         - If: not $.getAttr(deployed, false)
           Then:
           
The method ``.getAttr(attr_name, default_value)`` is defined in the class ``io.murano.Object``. An attribute is a (key,value) pair, which scope is particular instance of a class. 
It is not visible to a user.
Its value persists between deployments, ant it is very convenient to  store the application state in an attribute. 
In this exaple the application will be deployed only once. Later in the code you will see, that when the deployment is completed, 
the value of the attribuute ``deployed`` is set to ``true``.
If you will attempt to repeat the deployment of the SimpleVM application which is already deployed, the execution flow will not enter the ``If: not $.getAttr(deployed, false)`` 
for the second time
$.getAttr(name, default_value) - gets attribute from the Object model
$.setAttr(name, value) - store attribute value in the Object Model
Only basic types are supported (boolean, int, string)
         
::
   
             - $.environment.reporter.report($this, 'Creating a VM ')

This is a log method to send status log message back to UI
This is what users will see in the UI during deployment time 
The variable ``$this`` holds the reference to the current application being deployed.
This is how the UI knows, which status is update. 
.. todo:: provide the link to the picture with logs

Please note, the you will be unable to see any other logs on the UI except for these.
If error happens, Murano will not propagate error message to the UI.
That's why it is impotant to generate abundant log messages.


 :: 
   
             # Security groups object is a list of dicts [{}]
             # each dict element in this list should have following key:value entries:
             #  FromPort: value - lower boundary of the port range
             #  ToPort: value - upper boundary of the port range
             #  IpProtocol: <tcp|udp> - transport protocol type TP or UDP
             #  External: <true|false> - if true it opens for CIDR:0.0.0.0/0
             #                           if false it opens port only for SecGroup members
             - $securityGroupIngress:
                 - ToPort: 22
                   FromPort: 22
                   IpProtocol: tcp
                   External: true
             - $.environment.securityGroupManager.addGroupIngress($securityGroupIngress)
             
It is important to stress, that MuranoPL API has no service (utility) classes. Every class defined in MuranoPL API
represents some tangible entity.
MuranoPL approach is to pass a structure, not a service class. 
In the code sample above,  the method ``addGroupIngress($securityGroupIngress)`` expects array that contains one item, the item type is dictionary.
This method asks Heat to create a new SecurityGroup. The method returns when Secuity Group is created.

::

             # Now lets ask our instance to deploy itself.
             # Inside deploy method there are additional steps which configure networks for the environment
             - $.instance.deploy()

The variable ``$.instance`` is type Instance, it is defined here: http://murano.readthedocs.org/en/latest/murano_pl/core_classes.html#class-instance             
This method asks  Heat to create a a new Virtual Machine and returns whenmachine is created.

::

             - $.environment.reporter.report($this, 'The new VM is created')
             # Save host information to local variables
             - $.host: $.instance.ipAddresses[0]
             - $.user: 'root'

The code above sets OUT properties: ``host`` and ``user``. Please note the syntax difference between class properties and local variables:
``$.class_property_name`` vs. ``$local_variable_name``: class property has the prefix "."

 ::
 
             # Format log message based on variables values
             - $msg: format('{0}@{1}', $.user, $.host)
             - $.environment.reporter.report($this, 'The VM is available ' + $msg)

The code sample above is a very useful example of how to log some values to UI

 ::
 
             - $.setAttr(deployed, true) 
            
.. warning:: Beware of **True** and **true**. **True** is string literal, **true** is a boolean literal. ``If true`` evaluates to true, ``If false`` evaluates to false. Both ``If True`` and ``If False`` evaluate to true.



::
   
     restartVM:
       Usage: Action
       Body:
         - $.environment.reporter.report($this, 'Restarting the VM')
         # Create new Resources class to have an access to the package resources
         # Package resources are stored in Resource folder
         - $resources: new(sys:Resources)
         # Load yaml file with execution plan. Bind execution plan parameters with actual values.
         - $command: $resources.yaml('Restart.template').bind(dict(
             time => 'now'
             ))
         # Send execution plan to the agent on the instance
         - $.instance.agent.call($command, $resources)
         - $.environment.reporter.report($this, 'Restart command was sent to VM')
   
.. todo:: describe action method

**Resources/Restart.template**::

   FormatVersion: 2.0.0
   Version: 1.0.0
   Name: Restart VM
   
   Parameters:
     time: $time
   
   
   Body: |
     return restart(args.time).stdout
   
   Scripts:
     restart:
       Type: Application
       Version: 1.0.0
       EntryPoint: restart.sh
       Files: []
       Options:
         captureStdout: true
         captureStderr: true
      
      
Source code: https://github.com/gokrokvertskhov/murano-trainings/blob/master/Day_1/io.murano.apps.test.SimpleVM/Resources/Restart.template

Let's inspect it line-by-line:

.. todo:: add the section

**Resources/scripts/restart.sh**::

   #!/bin/bash
   shutdown -r $1

Source code: https://github.com/gokrokvertskhov/murano-trainings/blob/master/Day_1/io.murano.apps.test.SimpleVM/Resources/scripts/restart.sh

**UI/ui.yaml**::

   Version: 2
   
   Application:
     ?:
       type: io.murano.apps.test.SimpleVM
     name: $.appConfiguration.name
     instance:
       ?:
         type: io.murano.resources.LinuxMuranoInstance
       name: generateHostname($.instanceConfiguration.unitNamingPattern, 1)
       flavor: $.instanceConfiguration.flavor
       image: $.instanceConfiguration.osImage
       assignFloatingIp: $.appConfiguration.assignFloatingIP
       keyname: $.instanceConfiguration.keyPair
   
   Forms:
     - appConfiguration:
         fields:
           - name: name
             type: string
             label: Application Name
             initial: Tomcat
             description: >-
               Enter a desired name for the application. Just A-Z, a-z, 0-9, dash and
               underline are allowed
           - name: assignFloatingIP
             type: boolean
             label: Assign Floating IP
             description: >-
                Select to true to assign floating IP automatically
             initial: false
             required: false
             widgetMedia:
               css: {all: ['muranodashboard/css/checkbox.css']}
     - instanceConfiguration:
         fields:
           - name: title
             type: string
             required: false
             hidden: true
             description: Specify some instance parameters on which the application would be created
           - name: flavor
             type: flavor
             label: Instance flavor
             description: >-
               Select registered in Openstack flavor. Consider that application performance
               depends on this parameter.
             required: false
           - name: osImage
             type: image
             imageType: linux
             label: Instance image
             description: >-
               Select a valid image for the application. Image should already be prepared and
               registered in glance.
           - name: keyPair
             type: keypair
             label: Key Pair
             description: >-
               Select a Key Pair to control access to instances. You can login to
               instances using this KeyPair after the deployment of application.
             required: false
           - name: availabilityZone
             type: azone
             label: Availability zone
             description: Select availability zone where the application would be installed.
             required: false
           - name: unitNamingPattern
             label: Hostname
             type: string
             required: false
             widgetMedia:
               js: ['muranodashboard/js/support_placeholder.js']
               css: {all: ['muranodashboard/css/support_placeholder.css']}

Source code: https://github.com/gokrokvertskhov/murano-trainings/blob/master/Day_1/io.murano.apps.test.SimpleVM/UI/ui.yaml

Let's inspect it line-by-line:

.. todo:: add the section
   
How to Upload and Deploy Simple VM?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



Rename it to MyFirstVM  (unique name) and upload
   
   Update script (sh) and upload again
   Update template: Add logging 
   
   Local variables vs parameters
      lv -runtime , params - object model
   Update template:  Update workflow to report IP of vm

   Update Template: format function, new logging
   
   Note on syntax:
      whitespaces only,
      No tabs, etc.
   
.. todo:: How to transit to UI ???
