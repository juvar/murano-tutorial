.. _simple_vm_application_label:

Demo Application: Simple VM
---------------------------

What is Simple VM?
~~~~~~~~~~~~~~~~~~

.. todo:: create the section
   
https://github.com/gokrokvertskhov/murano-trainings/tree/master/Day_1/io.murano.apps.test.SimpleVM

How to Upload and Deploy Simple VM?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Simple VM Code Explained
~~~~~~~~~~~~~~~~~~~~~~~~

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

It is not MuranoPL, it is special format file. This is the analog of a manifest file you would see in a jar archave.

Let's inspect manifest.yaml line-by-line:

::

   Format: 1.0

Specifies the version of the format for manifest.yaml. Currently 1.0 is the only available version.


::

   Type: Application
   
| Specifies the type of the package.
| Please note, that the type name ``Application`` starts with the capital letter. This is the naming convention for all the predefined constants.
| The other available type is ``Type: Library``. Library is a package that contains classes common to multiple applications. 
| Take ``Core Library`` for example. See img ... 
.. todo:: add ref to the image

::
 
   FullName: io.murano.apps.test.SimpleVM
   
This is a globally unique name of a Murano application.   To ensure the global uniqueness, we follow the same naming convention as of Java.

::

   Name: Test SimpleVM
   
Display name of an application. You will be able to reset a display name when you upload SimpleVM package to Murano.


::

   Description: |  
    This is a simple test app with a single VM.
   Author: 'Mirantis, Inc'
Self-descriptive piece of code, but lets's take a note on the syntax.

* ``|`` 
* double white space start the line ``This is a simple test...``
.. todo:: add the note on yaml syntax 

.. warning:: Avoid TAB symbols inside yaml files. If yaml contains a TAB symbol, if will not be parsed correctly. The error message 

::

   Tags: [test] 
   
| This is array of categories, also known as tags. You can search an application by its category.
| You may want to specify several categories for one application ``Tags: [test, Web]``
| These are very same categories, you have seen on the screen ...
.. todo:: Add link to the image

::

   Classes:
    io.murano.apps.test.SimpleVM: SimpleVM.yaml
    
| This section is a mapping between all classes present in SimpleVM application and the file names
| where thease classes defined in. This is one-to-one relatioonship: i.e. one and only one class per file.
| The line ``io.murano.apps.test.SimpleVM: SimpleVM.yaml`` says that the class ``io.murano.apps.test.SimpleVM`` 
| is defined in the file ``SimpleVM.yaml``
| If you had another class, say, ``io.murano.apps.test.ComplexAction``defined in file ``ComplexAction.yaml``
| your Classes section would be

::

   Classes:
    io.murano.apps.test.SimpleVM: SimpleVM.yaml
    io.murano.apps.test.ComplexAction: ComplexAction.yaml
    
| If I may have multiple classes within my Application, what is the entry point?
| How Murano knows, which one is the "main" class?
| The "main" class is named exacly as the package FullName. In SimpleVM it is ``io.murano.apps.test.SimpleVM``
| Such a class must extend Murano Core class ``io.murano.Application``
| and it must implemen two method ``initialize`` and ``deploy`` 


See more details in the section:  :ref:`simple_vm_yaml_label` 
 
 
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

| Namespaces in Murano are like namespaces in XML.
| Default namespace ``=:`` 

::
   
   Name: SimpleVM 
   
| The name of the current class. Default namespace is ommitted.

::
      
   Extends: std:Application  
   
| This clause specifies, that class ``io.murano.apps.test.SimpleVM`` inherits from  ``io.murano.Application`` 
| MuranoPL allows multiple inheritance. In case if you class inhrerits from several classes, pass them as an array.
| Please note, that this code will cause parsing error ``Extends: [std:Application, base:MyBasicVM]`` bacause in contains ``:``
| Either pass the  fully-qualified class names as ``Extends: [io.murano.Application, io.murano.mybase.MyBasicVM]`` 
| or use ``-`` array syntax::
   
   Extends: 
     - std:Application
     - base:MyBasicVM  
   
| Then goes the section that describes class's properties.
| All properties are "public" - you can reach and read them outside the instance of SimpleVM class.
| There are various types of properties. In this example we have only ``In`` and ``Out`` types of properties.

| Let's take a look at the ``In`` properties: ``name`` and ``instance``. 
| We do not specify their type explicitly, so they gat default type which is ``In``

::
   
   Properties: 
     name:
       Contract: $.string().notNull() 
       
| The sign ``$`` denotes current context. In this case it is the ``name`` property of current instance of SimpleVM class   ??? == $this.name.string().notNull()
| The Contract clause enforces that the ``name`` is a non-empty string.
| This is how it works: 
| When the value of ``name`` propety will be passed to the SimpleVM instance, 
| the function ``string()`` will be called on this value. Every MuanoPL function either returns value, or throws error.
| string() function tryes to convert the value to string. If conversion is successfull, it returns the string value, 
| if not, it throws error.
| If string() returns a value, then notNull() funtion will be called on the string value.
| notNull() function throws error, if value is null, otherwise returns the value itself.

| ``instance`` property is a non-null objects, that represents Virtual VM (==aka instance you see in Murano dashboard)

::   

     instance:
       Contract: $.class(res:Instance).notNull()

| This contract ensures that instance parameter  is an instance of class Instance and that it is not null.       
| The function call ``class(res:Instance)`` attemps to convert ``instance`` to the object of type ``io.murano.resource.Instance``
| The MuranoPL object (i.e the instance of MuranoPL class) is a dictionary  of key-values pairs. Every object has a unique (string) id. 
| Object has a two representaions:
          -- as a dictionary somewhere in the object model
          -- as an id
 
| Object is represented as a dictionary only once  in the object model. The object that includes it is called OWNER.
| Anywhere else in the object model the object is referenced by its ID.
| Thus we have two independent object hierarchies: Inherinace and Owner, plus a graph of references between objects.
| Class has constructor new that creates instances of this class


::
     
     host:
       Contract: $.string()
       Usage: Out
   
     user:
       Contract: $.string()
       Usage: Out

| The properties ``host`` and  ``user`` are ``Out`` properties. The methods of this class will set value to them. 
| Let's take a look at the first of the methods, ``initialize``

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

| Use this method to initialize private properties of SimpleVM, that are not visible outside the particular instance of SimpleVM class.
| ``$.environment:`` is a declaration of private property
| The values of this property is returned by function .find(). It searches the owner object which type is Environment up in the OWNER hierarchy 
| Here ``$`` means the environment peoperty of current instance.
| It is the same as ``$this.environment`` 

Function ``require()`` verifies if ``find()`` function returns any value. If no value is returned, it throws ``error``. 


::
 
     deploy: 
       Body:
       
| The ``deploy`` method of your 'main' class in an entry point to your application.
| It is inherited from ``murano.io.Application``.
| The ``deploy`` method has no parameters, but MuranoPL methods may take parameters
.. todo:: add reference to the example of a method thattaked parameters

::

         - If: not $.getAttr(deployed, false)
           Then:
           
| The method ``.getAttr(attr_name, default_value)`` is defined in the class ``io.murano.Object``. An attribute is a (key,value) pair, which scope is particular instance of a class. 
| It is not visible to a user.
| Its value persists between deployments, ant it is very convenient to  store the application state in an attribute. 
| In this exaple the application will be deployed only once. Later in the code you will see, that when the deployment is completed, 
| the value of the attribuute ``deployed`` is set to ``true``.
| If you will attempt to repeat the deployment of the SimpleVM application which is already deployed, the execution flow will not enter the ``If: not $.getAttr(deployed, false)`` 
| for the second time
| $.getAttr(name, default_value) - gets attribute from the Object model
| $.setAttr(name, value) - store attribute value in the Object Model
| Only basic types are supported (boolean, int, string)
         
::
   
             - $.environment.reporter.report($this, 'Creating a VM ')

| This is a log method to send status log message back to UI
| This is what users will see in the UI during deployment time 
| The variable ``$this`` holds the reference to the current application being deployed.
| This is how the UI knows, which status is update. 
.. todo:: provide the link to the picture with logs

| Please note, the you will be unable to see any other logs on the UI except for these.
| If error happens, Murano will not propagate error message to the UI.
| That's why it is impotant to generate abundant log messages.


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
             
| It is important to stress, that MuranoPL API has no service (utility) classes. Every class defined in MuranoPL API
| represents some tangible entity.
| MuranoPL approach is to pass a structure, not a service class. 
| In the code sample above,  the method ``addGroupIngress($securityGroupIngress)`` expects array that contains one item, the item type is dictionary.
| This method asks Heat to create a new SecurityGroup. The method returns when Secuity Group is created.

::

             # Now lets ask our instance to deploy itself.
             # Inside deploy method there are additional steps which configure networks for the environment
             - $.instance.deploy()

| The variable ``$.instance`` is type Instance, it is defined here: http://murano.readthedocs.org/en/latest/murano_pl/core_classes.html#class-instance             
| This method asks  Heat to create a a new Virtual Machine and returns whenmachine is created.

::

             - $.environment.reporter.report($this, 'The new VM is created')
             # Save host information to local variables
             - $.host: $.instance.ipAddresses[0]
             - $.user: 'root'

| The code above sets OUT properties: ``host`` and ``user``. Please note the syntax difference between class properties and local variables:
|  ``$.class_property_name`` vs. ``$local_variable_name``: class property has the prefix "."

 ::
 
             # Format log message based on variables values
             - $msg: format('{0}@{1}', $.user, $.host)
             - $.environment.reporter.report($this, 'The VM is available ' + $msg)

| The code sample above is a very useful example of how to log some values to UI

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