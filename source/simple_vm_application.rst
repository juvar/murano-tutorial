.. _simple_vm_application_label:

Demo Application: Simple VM
---------------------------

.. todo:: Move css that fixes text width into css stylesheet

.. raw:: html

    <style> p {max-width:800px; width:100%; font-size:16px; color:rgb(0,50,100); text-align:left;}  
            strong {font-size:16px;}
            li {font-size:16px;}
            div {max-width:800px; width:100%;}
            h1 {font-size:26px; color: black; padding-top: 125px; padding-bottom: 125px;}
            h2 {font-size:24px; color: black; padding-top: 125px; padding-bottom: 125px;}
            h3 {font-size:22px; color: black; padding-top: 125px; padding-bottom: 125px;}
            pre.sample {background-color:#F0F0F0;}
                
    </style>



What is Simple VM?
~~~~~~~~~~~~~~~~~~

This tutorial uses a demo application Simple VM to demonstrate how create you own Murano Application.
This section will walk you through its source code line-by-line and then explain how to upload, deploy and test it.

Simple VM is a very basic application: it creates instance and provides an action that can restart it trough Murano Dashboard.

The source code of Simple VM is available in github:   
https://github.com/gokrokvertskhov/murano-trainings/tree/master/Day_1/io.murano.apps.test.SimpleVM

.. topic:: Instantaneous Terminology. 
   SimpleVM has a source code written in MuranoPL. This programming language is object-oriented and we will see ``classes``, 
   ``objects`` and ``object instances``. To distinguish between an object instance (created when MuranoPL of  SimpleVM is executed), 
   and an instance of a virturual machine (created when SimpeVM application is deployed) let's agree on the following naming convention:
   Whenever *instance* is used, it denotes an object instance, which is created when MuranoPL of  SimpleVM is executed.
   To reference an instance of a virturual machine, which is created when SimpeVM application is deployed, 
   we will use the term *VM instance*.
   
   Also the source code below has a variable called ``$.instance``. To distinguish it from other instances this tutorial will reference it as 
   *variable instance* or *property instance*


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
   
   
The structure of the typical Murano Application is described here: :ref:`what_is_inside_package_label`

The unusual part of Simple VM is that it does not install any software on the instance it creates.
That's why there is no any .sh script that would install software and consequently there is no template for install script in Resouces folder.
On the other hand, Simple VM provides a restart action, which is available at Murano Dashboard.
The logic of Restart action is placed in Restart.template and restart.sh.  

manifest.yaml
_____________

**manifest.yaml**

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

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
It has a fixed format based on YAML (this is not MuranoPL).

Let's inspect manifest.yaml line-by-line:

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

   Format: 1.0

The field **Format** specifies the version of the format for manifest.yaml. Currently 1.0 is the only available version.


.. code-block:: yaml
   :linenos:
   :lineno-start: 2   


   Type: Application
   
The field **Type** specifies the type of the package. There are only two possible values: ``Application`` and ``Library``.
The value ``Application`` is used for all Murano Applictions. The value ``Library`` is a special case. 
It is used for a package that contains classes common to multiple applications.
The example of a library, is ``Core Library``. It contains core Murano API (http://murano.readthedocs.org/en/latest/murano_pl/core_classes.html) 
You can see Core Library on the Packages Panel:

.. image:: images/packages.png

Note, that both values: ``Application`` and ``Library`` start with capital letter. This is the naming convention for all pre-defined values in Murano code.

.. code-block:: yaml
   :linenos:
   :lineno-start: 3   

 
   FullName: io.murano.apps.test.SimpleVM
   
**FullName** is a globally unique name of a Murano application. 
To ensure the global uniqueness, we follow the same naming convention as the naming convention of Java packages and classes.
The "io.murano.apps.test." is a "package" part of the name.
"SimpleVM" is a "class" part of the name.


.. topic:: Java Language Specification states: 
  
  You form a unique package name by first having (or belonging to an organization that has) an Internet domain name, such as oracle.com.
  You then reverse this name, component by component, to obtain, in this example, com.oracle, and use this as a prefix for your package names, 
  using a convention developed within your organization to further administer package names.
   
  Such a convention might specify that certain package name components be division, department, project, machine, or login names. 
  
  The first component of a unique package name is always written in all-lowercase ASCII letters and should be one of the top level domain names, 
  such as com, edu, gov, mil, net, or org, or one of the English two-letter codes identifying countries as specified in ISO Standard 3166. 
  
  The name of a package is not meant to imply where the package is stored on the Internet. 
  
  The suggested convention for generating unique package names is merely a way to piggyback a package naming convention on top of an existing, 
  widely known unique name registry instead of having to create a separate registry for package names. 
  
  [...] 
  
  Names of class types should be descriptive nouns or noun phrases, not overly long, in mixed case with the first letter of each word capitalized.


.. code-block:: yaml
   :linenos:
   :lineno-start: 4   


   Name: Test SimpleVM
   
The field **Name** is a display name of an application. You will be able to reset a display name when you upload SimpleVM package to Murano.


.. code-block:: yaml
   :linenos:
   :lineno-start: 5   


   Description: |  
    This is a simple test app with a single VM.
   Author: 'Mirantis, Inc'

The field **Description** is an application decription rendered on the application tile. Lets's take a closer look on the syntax.

* ``|`` - vertical line symbol comes from YAML syntax. It means that all the leading indents and new line symbols should be preserved. This is very useful for long, multi-line descriptions, because this affects how they are desplayed on the UI.
* double white space start the line ``This is a simple test...``  It will be preserved when description is rendered on the Murano Dashboard.
.. todo:: test this 

.. warning:: Avoid TAB symbols inside YAML files. If YAML contains a TAB symbol, if will not be parsed correctly. The error message may be cryptic or misleading

.. code-block:: yaml
   :linenos:
   :lineno-start: 8   

   Tags: [test] 
   
**Tags** field  is an array of tags. You can search an application by its tag.
You may want to specify several tags for one application ``Tags: [test, Web]``
YAML offers two equivalent syntaxes to write an array

.. raw:: html

   <table>
   <tr>
   <td>
   <pre class="sample">
   Tags: [test, Web]
   </pre>
   </td>
   </tr>
   </table>

is the same as

.. raw:: html

   <table>
   <tr>
   <td>
   <pre class="sample">

   Tags:
     - test
     - Web

   </pre>
   </td>
   </tr>
   </table>
     

.. code-block:: yaml
   :linenos:
   :lineno-start: 9

   Classes:
    io.murano.apps.test.SimpleVM: SimpleVM.yaml
    
The field **Classes** is a mapping between all classes present in SimpleVM application and the file names
where these classes defined in. This is one-to-one relatioonship: i.e. one and only one class per file.
The line ``io.murano.apps.test.SimpleVM: SimpleVM.yaml`` says that the class ``io.murano.apps.test.SimpleVM`` 
is defined in the file ``SimpleVM.yaml``
If you had another class, say, ``io.murano.apps.test.Utils``defined in file ``Utils.yaml``
your Classes section would be

.. raw:: html

   <table>
   <tr>
   <td>
   <pre class="sample">

   Classes:
    io.murano.apps.test.SimpleVM: SimpleVM.yaml
    io.murano.apps.test.Utils: Utils.yaml
   </pre>
   </td>
   </tr>
   </table>
    
From syntax point of view, this is the example of YAML Associative array aka dictionary. 
Similar to array, it has one-line (== inline) syntax

.. raw:: html

   <table>
   <tr>
   <td>
   <pre class="sample">

   Classes: {io.murano.apps.test.SimpleVM: SimpleVM.yaml, io.murano.apps.test.Utils: Utils.yaml}

   </pre>
   </td>
   </tr>
   </table>    
        

If I may have multiple classes within my Application, what is the entry point?
How Murano knows, which one is the "main" class?
The answer is that the "main" class is named exacly as the package *FullName*. In SimpleVM it is ``io.murano.apps.test.SimpleVM``

The main class must explicitly extend core Murano class ``io.murano.Application``
and it must implement two methods ``initialize`` and ``deploy`` 


See more details about Murano Classes in the section:  :ref:`simple_vm_yaml_label` 
 
 
.. _simple_vm_yaml_label:

Classes/SimpleVM.yaml
_____________________

**Classes/SimpleVM.yaml**

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

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
             # Inside deploy method there are additional steps which configure networks
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

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

   Namespaces:
     =: io.murano.apps.test
     std: io.murano
     res: io.murano.resources
     sys: io.murano.system
     apps: io.murano.apps

Namespaces in Murano are like namespaces in XML.
The symbol ``=:`` denotes a default namespace. 
In the code snippet above it means "if a namespace is not specified, assume that it is ``io.murano.apps.test`` "  
For example, ``Name: SimpleVM`` is equivalent to ``Name: io.murano.apps.test.SimpleVM``, 
because the default namespace is used in place of omitted one.

Non-default namespace is specified as mapping ``<namespece short alias>: <package full name>``. For example, 
``std: io.murano`` means "assume the namespace is ``io.murano`` whereever ``std:`` is used". 
Thus, ``Extends: std:Application`` is equivalent to ``Extends: io.murano.Application`` 

You may not declare a namespace and use a fully-qualified class names instead.

.. code-block:: yaml
   :linenos:
   :lineno-start: 10   
   
   Name: SimpleVM 
   
The name of the class defined in this YAML file. Neither the namespace, nore full package name is specified.
That's why the default package is used. The full name of the class is ``io.murano.apps.test.SimpleVM``.

.. code-block:: yaml
   :linenos:
   :lineno-start: 17   
      
   Extends: std:Application  
   
This clause specifies, that class ``io.murano.apps.test.SimpleVM`` inherits from  ``io.murano.Application`` 
MuranoPL allows multiple inheritance. In case if you class inhrerits from several classes, pass them as an array.
There are two possible syntax notations for an array.
The first one is "one line" syntax:

.. raw:: html

   <table>
   <tr>
   <td>
   <pre class="sample">
   Extends: [io.murano.Application, io.murano.mybase.MyBasicVM]
   </pre>
   </td>
   </tr>
   </table>

It is important to pass fully-qualified class names, because colon ``:`` symbol used in namespace will cause parsing error.  

Do **not** do this: ``Extends: [std:Application, base:MyBasicVM]``.

The second syntax option is to use "multi-line" array syntax. 
Every array element is put on separate line, with proper indent and prefixed by the symbol: ``-``: 

.. raw:: html

   <table>
   <tr>
   <td>
   <pre class="sample">   
   Extends: 
     - std:Application
     - base:MyBasicVM  
   </pre>
   </td>
   </tr>
   </table>
   

Multi-line syntax allows using both namespace prefixes (i.e. ``std:Application``) and fully-qualified names (i.e. ``io.murano.Application``)

.. code-block:: yaml
   :linenos:
   :lineno-start: 19   

   
   Properties: 
     name:
       Contract: $.string().notNull()  
       

This section describes class properties.
All properties are "public" - you can access them and get their values outside an instance of SimpleVM class.


There are various types of properties. In this example we have only ``In`` and ``Out`` types of properties.

Let's take a look at the properties ``name`` and ``instance``. They are ``In`` properties. 
The type ``In`` is default. You do not have to specify it explicitely.

``In`` properties receive their value fom outside the class. The ``Contract`` clause specifies the expected
type of a passed value and the its value range. Let's take a closer look at it. 

The sign ``$`` denotes current context. In this case it is the ``name`` property of current instance of SimpleVM class.

..   ??? == $this.name.string().notNull()

The Contract clause enforces that the value of ``name`` property is a non-empty string.
This is how it works: 
When the value of ``name`` propety is passed to the SimpleVM instance, 
the function ``string()`` is called on this value. Every MuranoPL function either returns value, or throws error.
``string()`` function tryes to convert the value to string. If conversion is successfull, it returns the string value, 
if not, it throws error.
If ``string()`` returns a value, then ``notNull()`` function will be called on the string value.
``notNull()`` function throws error, if a value is null, otherwise returns the value itself.


.. code-block:: yaml
   :linenos:
   :lineno-start: 23      

     instance:
       Contract: $.class(res:Instance).notNull()

This contract ensures that instance parameter  is an instance of class Instance and that it is not null.       
Class ``Instance`` is an abstruction that represents an instance of virtual machine.

The function call ``class(res:Instance)`` attemps to convert ``instance`` to the object of type ``io.murano.resource.Instance``
The MuranoPL object (i.e the instance of MuranoPL class) is a dictionary  of key-values pairs. Every object has a unique (string) id. 
Object has a two representaions:

* as a dictionary somewhere in the object model
* as an id
 
Object is represented as a dictionary only once  in the object model. The object that includes it is called OWNER.
Anywhere else in the object model the object is referenced by its ID.
Thus we have two independent object hierarchies: Inheritance and Owner, plus a graph of references between objects.
Class has constructor ``new`` that creates instances of this class


.. code-block:: yaml
   :linenos:
   :lineno-start: 26   
     
     host:
       Contract: $.string()
       Usage: Out
   
     user:
       Contract: $.string()
       Usage: Out

The properties ``host`` and  ``user`` are ``Out`` properties. The methods of this class will set value to them. 

Let's take a look at the first of the methods, ``initialize``

.. code-block:: yaml
   :linenos:
   :lineno-start: 36   
 
   Methods:
     initialize:
       Body: 
         - $.environment: $.find(std:Environment).require() 


This is a "constructor" of SimpleVM class. The lifecycle of a SimpleVM object is:

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


.. code-block:: yaml
   :linenos:
   :lineno-start: 42
 
     deploy: 
       Body:
       
The ``deploy`` method of your 'main' class in an entry point to your application.
It is inherited from ``murano.io.Application``.
The ``deploy`` method has no parameters, but MuranoPL methods may take parameters.

.. todo:: add reference to the example of a method that takes parameters

.. code-block:: yaml
   :linenos:
   :lineno-start: 50   
    
         - If: not $.getAttr(deployed, false)
           Then:
           
The method ``.getAttr(attr_name, default_value)`` is defined in the class ``io.murano.Object``. 
An attribute is a (key,value) pair, which scope is a particular instance of a class. 
It is not visible to a user.
Its value persists between deployments, and it is very convenient to store an application state in its attributes. 
In this example the application will be deployed only once. Later in the code you will see, that when the deployment is completed, 
the value of the attribuute ``deployed`` is set to ``true``.
If you will attempt to repeat the deployment of the SimpleVM application which is already deployed, the execution flow will not enter the ``If: not $.getAttr(deployed, false)`` 
for the second time.

``$.getAttr(name, default_value)`` - gets attribute from the Object model

``$.setAttr(name, value)`` - store attribute value in the Object Model

Only basic types are supported (boolean, int, string)
         
.. code-block:: yaml
   :linenos:
   :lineno-start: 55   
        
             - $.environment.reporter.report($this, 'Creating a VM ')

This is a log method to send status log message back to UI.
This is what users will see in the UI during deployment time.
The variable ``$this`` holds the reference to the current application being deployed.
This is how the UI knows, which status is update.

.. todo:: provide the link to the picture with logs

Please note, the you will be unable to see any other logs on the UI except for these.
If error happens, Murano will not propagate error message to the UI.
That's why it is impotant to generate abundant log messages.


.. code-block:: yaml
   :linenos:
   :lineno-start: 57   
   
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
MuranoPL approach is to pass a structure, not a utility class. 
In the code sample above,  the method ``addGroupIngress($securityGroupIngress)`` expects array that contains one item, the item type is dictionary.
This method asks Heat to create a new SecurityGroup. The method returns when Secuity Group is created.


.. warning:: Current version of MuranoPL does not support Egress groups.

.. code-block:: yaml
   :linenos:
   :lineno-start: 70
     
             # Now lets ask our instance to deploy itself.
             # Inside deploy method there are additional steps which configure networks
             - $.instance.deploy()

The variable ``$.instance`` is type Instance, it is defined here: http://murano.readthedocs.org/en/latest/murano_pl/core_classes.html#class-instance             
This method asks  Heat to create a a new Virtual Machine and returns when machine is created.

.. code-block:: yaml
   :linenos:
   :lineno-start: 73 
     
             - $.environment.reporter.report($this, 'The new VM is created')
             # Save host information to local variables
             - $.host: $.instance.ipAddresses[0]
             - $.user: 'root'

The code above sets OUT properties: ``host`` and ``user``. Please note the syntax difference between class properties and local variables:
class property has the prefix "." : ``$.class_property_name`` vs. ``$local_variable_name``.

An instance variable has a property ``ipAddresses``. Its type is an array, as a vm instance may have several IP addresses.
In this example we get any of them.

.. todo:: Check the type.

.. todo:: How to output all the IP addresses?

.. todo:: Can I reset a parameter?

.. code-block:: yaml
   :linenos:
   :lineno-start: 77   

             # Format log message based on variables values
             - $msg: format('{0}@{1}', $.user, $.host)
             - $.environment.reporter.report($this, 'The VM is available ' + $msg)

The code sample above is a very useful example of how to log some values to UI, in contrast to logging fixed String message.

The statement ``$msg:`` declares a local variable. Here ``$`` denotes a current context (i.e. a particular instance), 
``msg`` is a name of a local variable, ``:`` is an assignment operator. It is followed by an initial value of a variable.
In this example the initial value is not a mere constant, but a function call. The function ``format`` is one of the available functions in MUranoPL.
The full list of all functions is available here:

.. todo:: add  the reference to the list of all functions.

The first parameter of ``format`` function is a string that represents the pattern of the message. 
It may contain an arbitrary text and inclusions of curly braces that surround a number, such as ``{0}`` and ``{1}``.
A number in curly braces is a placeholder where a parameter value will be placed.
The second parameter of a function ``format`` and all other parameters that follow it will be used in place of a placeholder.
The numbering starts from ``0``. So, ``$.user`` is a parameter number ``0``, ``$.host`` is a parameter number ``1``.
The value of an instance property ``$.user``  will substitute ``{0}``, ``$.host`` will substitute ``{1}``.
In this example the outout will be similar to ``root@172.10.10.10``

.. todo:: Can I reassign a value to a variable?

.. todo:: Does vaiable preserve its value between method calls?

.. code-block:: yaml
   :linenos:
   :lineno-start: 80   
 
             - $.setAttr(deployed, true) 

This code sets the attribute of current instance called ``deployed`` to ``true``. 
This attribute indicates, that the application was deployed successfully.
Please note, that this particular ``deploy`` method is written in such a way, that deplyment logic
is executed only once. (Please see ``if`` statement on the line #) 

In the more sophisticated application you may allow setting some extra configuration parameters 
and doing incremental deploy logic that modifies vm instances and their spftware according to 
new deployment configuation.

.. todo:: this of the example
            
.. warning:: Beware of **True** and **true**. **True** is string literal, **true** is a boolean literal. ``If true`` evaluates to true, ``If false`` evaluates to false. Both ``If True`` and ``If False`` evaluate to true.

Now let's take a look the Action that restarts VM instance.

.. code-block:: yaml
   :linenos:
   :lineno-start: 82   
        
     restartVM:
     
Action is defined in a separate method.  ``restartVM:`` is a method name.


.. code-block:: yaml
   :linenos:
   :lineno-start: 83   
        
       Usage: Action
       
The ``Usage`` clause identifies that the method ``restartVM`` defines an action

.. code-block:: yaml
   :linenos:
   :lineno-start: 84   
        
       Body:
         - $.environment.reporter.report($this, 'Restarting the VM')
         
The method body starts with ``Body`` clause. There is no closing close.
All is defined by leading indents. 

.. code-block:: yaml
   :linenos:
   :lineno-start: 86   
        
         # Create new Resources class to have an access to the package resources
         # Package resources are stored in Resource folder
         - $resources: new(sys:Resources)

Instantiating an object of class sys:Resources. 

.. todo:: give the link to the definition of the Resource class
 
.. code-block:: yaml
   :linenos:
   :lineno-start: 89   
        
         # Load yaml file with execution plan. Bind execution plan parameters with actual values.
         - $command: $resources.yaml('Restart.template').bind(dict(
             time => 'now'
             ))
             

``$resources.yaml('Restart.template')`` 
The ``yaml`` function reads a resource file 'Restart.template' and returns it in YAML format.

You may take a look to ``sys:Resources`` class (https://github.com/stackforge/murano/blob/master/meta/io.murano/Classes/system/Resources.yaml) 
and see that is almost empty and has no ``yaml`` function defined. Then where it does come from?

io.murano.system.Resources (and all other system classes) are implemented in Python. 
The system classes (including io.murano.system.Resources) you see in MuranoPL are mere placeholders.

The Python class is called ResourceManager and is available on github
here: https://github.com/stackforge/murano/blob/master/murano/engine/system/resource_manager.py

All other system classes are available on github next to ResourceManager:
https://github.com/stackforge/murano/blob/master/murano/engine/system/

If a system class has a method whicn name does not start with underscore symbol '_', 
then this method is availabe in MuranoPL code.
Thus,  Resources class offers ``string`` and ``json`` functions in addition to ``yaml``.
``string`` returns a resource as a string, ``json``  - as a json structure.


Now let's take a look at ``bind(dict (time => 'now'))`` function from inside to out.

Function ``dict`` is a YAQL (http://yaql.readthedocs.org/en/latest/) function that takes text input and constructs Python dictionary.
For the input ``time => 'now'`` the result is Python dictionary ``{time: 'now'}``

.. todo:: How to pass several parameters?

Function ``bind`` is another YAQL function. It searches YAML data for expressions ``$<param_name>``, such as ``$time`` and replaces them 
with corresponding values taken from the dictionary. In our example it will replace all ocurances of ``$time`` with ``now``.

You may read about YAQL here: (http://yaql.readthedocs.org/en/latest/)
The function ``dict`` is a build-in YAQL function.
The full list of build-in functions can be found here: https://github.com/stackforge/yaql/blob/stable/0.2/yaql/functions/builtin.py

The function ``bind`` is a Murano extention to YAQL functions. 
All Murano extentions are defined here: https://github.com/stackforge/murano/blob/master/murano/engine/system/yaql_functions.py


.. code-block:: yaml
   :linenos:
   :lineno-start: 93   
        
         # Send execution plan to the agent on the instance
         - $.instance.agent.call($command, $resources)

.. todo:: agent.call


.. code-block:: yaml
   :linenos:
   :lineno-start: 95   
        
         - $.environment.reporter.report($this, 'Restart command was sent to VM')
   
.. todo:: describe action method

**Resources/Restart.template**

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   
     
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

**Resources/scripts/restart.sh**


.. code-block:: bash
   :linenos:
   :lineno-start: 1   

   #!/bin/bash
   shutdown -r $1

Source code: https://github.com/gokrokvertskhov/murano-trainings/blob/master/Day_1/io.murano.apps.test.SimpleVM/Resources/scripts/restart.sh

Please note that Murano-agent executes bash scripts as a root user. This is a potential security breach, so do take measures to prevent 
malitious code execution. Do not upload Murano packages of unknown or dubious origin.

**UI/ui.yaml**


.. code-block:: yaml
   :linenos:
   :lineno-start: 1   
   
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
             description: >-
               Specify some instance parameters on which the application would be created
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
