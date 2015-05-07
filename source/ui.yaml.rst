.. _ui_yaml_explained_label:

UI YAML Explained Line-by-Line
******************************

.. ##todo:: Move css that fixes text width into css stylesheet


.. raw:: html

    <style> p {max-width:700px; width:100%; font-size:16px; color:rgb(0,50,100); text-align:left;}  
            strong {font-size:16px;}
            li {font-size:16px;}
            div {max-width:700px; width:100%;}
            h1 {font-size:26px; color: black; padding-top: 125px; padding-bottom: 125px;}
            h2 {font-size:24px; color: black; padding-top: 125px; padding-bottom: 125px;}
            h3 {font-size:22px; color: black; padding-top: 125px; padding-bottom: 125px;}
            pre.sample {background-color:#F0F0F0;}
                
    </style>



Let's inspect the source code line-by-line:

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

   Version: 2

The ``Version`` clause is the version of format used to specify UI.
The version 2 is the latest one. If you use Murano version 0.4 or older, specify version 1.
 
   
.. code-block:: yaml
   :linenos:
   :lineno-start: 3   

   Application:
       
The ``Application`` clause describes the object model of the application.
Murano-dashboard will use this model to generate JSON model that it will send to Murano-engine.

Murano JSON API to add a new Application to environment is described here: http://murano.readthedocs.org/en/latest/specification/#post-applications
Murano-dashboard generates JSON code based on the UI.yaml and on the values entered by a user.

This code will be similar to:

.. raw:: html

   <table>
   <tr>
   <td>
   <pre class="sample">   
      {
        "instance": {
          "flavor": "m1.medium",
          "image": "clod-fedora-v3",
          "?": {
            "type": "io.murano.resources.Instance",
            "id": "bce8308e-5938-408b-a27a-0d3f0a2c52eb"
          },
          "name": "nhekhv6r7mhd4"
        },
        "name": "sdf34sadf",
        "?": {
          "_26411a1861294160833743e45d0eaad9": {
            "name": "Telnet"
          },
          "type": "io.murano.apps.linux.Telnet",
          "id": "190c8705-5784-4782-83d7-0ab55a1449aa"
        }
      }
   </pre>
   </td>
   </tr>
   </table>
   
.. ##todo:: add actual JSON for SimpleVM

.. code-block:: yaml
   :linenos:
   :lineno-start: 4   

     ?:
       type: io.murano.apps.test.SimpleVM

The property ``type`` specifies the name of the main class of the application.
Please note that it starts with question mark symbol ``?``, which denotes system property.

.. #todo:: Define what is system properties

As you remember, SimpleVM class has two ``IN`` properties: ``name`` and ``instance``. UI form constructs their values based user input.
Now we will see how it happens.  

.. code-block:: yaml
   :linenos:
   :lineno-start: 6   

     name: $.appConfiguration.name
     

The property ``name`` corresponds to the SimpleVM IN property ``name``. It specifies the name of the application that will be rendered on the list of the applications added in the environment. 
This property takes a value ``$.appConfiguration.name``. Let's take a look at it. It is a YAQL expression. Any YAQL expression starts with ``$.``.
Property ``appConfiguration`` represents a UI form appConfiguration (it is defined later in the code) that contains the values you have specified when you added Simple VM appliation
to the environment. 

appConfiguration.name represents the value you enter in the name field.

This is the first screen

.. #todo:: add screenshot

.. code-block:: yaml
   :linenos:
   :lineno-start: 7   

     instance:
       ?:
         type: io.murano.resources.LinuxMuranoInstance
         
The instance clause defines what instance object will be created and passed to the deploy method of main class. 
It specifies the type of the instance and the values of its properties.
Please note that the type property is a system property, and it is prefixed with ?
 It is a yaql expression.

.. #todo:: Source Code of io.murano.resources.LinuxMuranoInstance is available here

.. warning:: The older versions of Murano require instance type io.murano.resources.Instance. Check deployment logs for error message if deploy fails.

.. #todo:: Add screenshot of the error

.. code-block:: yaml
   :linenos:
   :lineno-start: 10   

       name: generateHostname($.instanceConfiguration.unitNamingPattern, 1)
       
generateHostname is a yaql function that generates a hostname by a pattern.
It is available on github.

.. #todo:: cannot find it in Murano 

instanceConfiguration is a efinrd inside  a Form clause, which is defined below. Similarly to appConfiguration, it specifies the input paameters for the application.

.. code-block:: yaml
   :linenos:
   :lineno-start: 11  

       flavor: $.instanceConfiguration.flavor
       
Flavor property of instanceConfiguration defines a size of a vm. 

.. code-block:: yaml
   :linenos:
   :lineno-start: 12   
       
       image: $.instanceConfiguration.osImage
       
Image property of instanceConfiguration defines wich OS image will be installed on the vm

.. code-block:: yaml
   :linenos:
   :lineno-start: 13   

       assignFloatingIp: $.appConfiguration.assignFloatingIP

Boolean property of instanceConfiguration  - property that defines if an instance will have a floating IP address that makes it accessible from outside.

.. #todo:: How an instance receives it? There is no explicit assignment

.. code-block:: yaml
   :linenos:
   :lineno-start: 14   

       keyname: $.instanceConfiguration.keyPair
       
The name of a key pair. you need to specify it to be able to ssh and login onto the machine.

.. code-block:: yaml
   :linenos:
   :lineno-start: 16   
   
   Forms:
   
Forms clause contains an array of UI forms. Each of them will be displayed on the UI as a separate pop-up window there you will be able to enter values for their parameters.

.. code-block:: yaml
   :linenos:
   :lineno-start: 17   

     - appConfiguration:
     
This is the name of the first UI form. This form can be referenced by this name from YAQL expression as $.appConfiguration 

.. code-block:: yaml
   :linenos:
   :lineno-start: 18   

         fields:
    
fields property contains an array of fields. Each form contains an array of input fields where a user can type input values in.

.. code-block:: yaml
   :linenos:
   :lineno-start: 19   

           - name: name
           
The name of the field. A field may be referenced by its name in a yaqk expression.
This particular field may be referenced as $.appConfiguration.name

.. code-block:: yaml
   :linenos:
   :lineno-start: 20   
 
             type: string
             
The type of the field.

.. #todo:: The available values are

.. code-block:: yaml
   :linenos:
   :lineno-start: 21   

             label: Application Name
             
Text label that will displayed next to this field.

.. code-block:: yaml
   :linenos:
   :lineno-start: 22   

             initial: Tomcat
             
The filed will be pre-populatedwith this initial value. User may edit and replace it.

.. code-block:: yaml
   :linenos:
   :lineno-start: 23   

             description: >-
               Enter a desired name for the application. Just A-Z, a-z, 0-9, dash and
               underline are allowed
               
Help message that will be displayed to a user when a user puts cursor in its field. 
Please note that symbol >- instructs that a newline symbils must be converted into whitespaces.
Thus the desccription will be folded in a one line.

.. code-block:: yaml
   :linenos:
   :lineno-start: 26   
 
           - name: assignFloatingIP
             type: boolean
             label: Assign Floating IP
             description: >-
                Select to true to assign floating IP automatically
             initial: false
             required: false
             widgetMedia:
               css: {all: ['muranodashboard/css/checkbox.css']}
               
assignFloatingIP is anthother fiels. It is very similar to name field, excapt for two fields.

.. code-block:: yaml
   :linenos:
   :lineno-start: 32   

             required: false

Non-obligatory field. If the property requiref is ommitted, its default valyede is true. the field is required by default.

.. code-block:: yaml
   :linenos:
   :lineno-start: 33   

             widgetMedia:
               css: {all: ['muranodashboard/css/checkbox.css']}

This is the way to create a checkbox on the UI form. 
``widgetMedia`` represents Django widget. 
The code ``css: {all: ['muranodashboard/css/checkbox.css']}`` is the way to customize the look-and-feel of Django widget.
This code re-uses Django syntax described here: https://docs.djangoproject.com/en/1.8/topics/forms/media/
Django widgetrs are described here: https://docs.djangoproject.com/en/1.7/ref/forms/widgets/
All the available css are placed on github: 
https://github.com/openstack/murano-dashboard/tree/master/muranodashboard/static/muranodashboard/css

https://github.com/openstack/murano-dashboard/tree/master/muranodashboard/static/muranodashboard/js

.. #todo:: The other available values are: 

.. code-block:: yaml
   :linenos:
   :lineno-start: 35  

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
               
It is symmetrical to appConfiguration form except for it uses extra fields types  and widgetMedia.
type.

.. #todo:: hidden: true

