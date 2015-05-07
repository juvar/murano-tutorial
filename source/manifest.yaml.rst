.. _manifest_yaml_explained_label:

Manifest YAML Explained Line-by-Line
************************************

.. #todo:: Move css that fixes text width into css stylesheet


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


.. topic:: Namng Convention Taken From Java Language Specification: 
  
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
    
From syntax point of view, this is the example of YAML *associative array*, also named *dictionary*. 
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


**How to update manifest.yaml if I want to modify SimpleVM to install Tomcat?**

You will have to update ``Description:`` clause to reflect that the application installs Tomcat.
And you will have to update ``Tags:``: replace ``[test]`` with any descriptive tag or tags of your choise, such as  ``[Web]`` or ``[Web, Tomcat]``
   