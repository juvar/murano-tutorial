.. _restart_template_explained_label:

Restart Template Explained Line-by-Line
***************************************

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



Let's inspect it line-by-line:

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   


   FormatVersion: 2.0.0

The version of the format of restart template file. Currently available versions are 1.0.0, 2.0.0, 2.1.0.
1.0.0 is a historic version used for windows murano agents.
2.0.0 is a current version
2.1.0 is an upcoming version.It sould be used if resource template will be pass to a newest murano-agent that makes uses of latest kilo release functionality such as chef, puppet and downloadable files.  

.. warning:: The rest of the fields depend on the FormatVersion may change drastically formversion to version. This tutorial is based on version 2.0.0 and describes its format only.

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

   Version: 1.0.0

This is the version of the file itself. This is optional clause. Its only person is to inform a developer what is the vesion of resource.
So if you are updating your resource temple, it is a recommended to increment the version.

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

   Name: Restart VM

This is the name of the resource template. Again, its sole purpose is provide the information for developer.

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   
   
   Parameters:
     time: $time

Parameter clause defines parametes received by the resource template. The syntax is:   

.. raw:: html

   <table><tr><td><pre class="sample">

   Parameters:
     param1: $value1
     param2: $value2
     ...

     </pre></td></tr></table> 
      
.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

   Body: |
     return restart(args.time).stdout
     
The Body clause defines, which scripts should be executed and what resut should be returned.
The text that follows ``Body: |`` is python code. The bode of the ``Body`` a python function.

The code ``return restart(args.time).stdout`` means:

* call restart function. Restart function is transparently available due to declarations made in ``Scripts`` clause. (See below)
* pass ``time`` parameter to restart function
* capture the output to ``stdout`` made by scripts that are executed inside ``restart()`` function and return it as a result.

So what can I return from  a  ``Body`` function? 
It can be any object that can be serialized to json by a murano-agent. This can be:
* strings
* numbers
* literals True/False/None
* arrays
* dictionaries
* any combination of the above

Murano-agent would capture the returned object, serialize it to json and send it back to Murano-engine.
This works as an RPC call.

For example, you may return an object from your template as: 

.. raw:: html

   <table><tr><td><pre class="sample">
    
    Body: |
      return {
        'cpu': int(getCpuCoresCount().stdout) * 100,
        'memory': int(getMemoryBytes().stdout)
      }
      
      
  </pre></td></tr></table>
  
and then you can access it from a calling class as ``$<result_variable_name>.cpu`` and ``$<result_variable_name>.memory``.
Check out the sample code below:

.. raw:: html

   <table><tr><td><pre class="sample">
    
    - $resources: new(sys:Resources)
    - $template: $resources.yaml('my.template').bind(...)
    - result: $.instance.agent.call($template, $resources)
    - $.environment.reporter.report($this, 'The CPU usage is ' + $result.cpu) 
    
    
    </pre></td></tr></table>

Now let's take a look at what makes function ``restart()`` available from ``Body`` function.

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   
  
   Scripts:
   
``Scripts`` clause define what executable scripts are available as functions from ``Body`` function.
Its structure is:

.. raw:: html

   <table><tr><td><pre class="sample">

   Scripts:
     functionName1:
        ... more declarations here ...
     functionName2:
        ... more declarations here ...
     etc.   

    </pre></td></tr></table>

Anything that is defined in Scripts section the same way as ``functionName1`` and ``functionName2`` are availabe as 
python functions ``functionName1()``, ``functionName2()``, etc. from ``Body`` function.

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   
        
     restart:

The script defined in ``restart:`` section is available as ``restart()`` function in ``Body`` function.

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

       Type: Application
       
This clause defines the type of the executable script. This can be any script executable by shell. 
The other available values are: ``Chef`` and ``Puppet``. They were added in kilo release.
They are beyonf this tutoral.

.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

       Version: 1.0.0
    
This is the version of the script listed in ``EntryPoint`` (see below). The purpose on this clause is only to inform a developer.
 
.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

       EntryPoint: restart.sh
       
This is the enty point for the ``restart`` function. This can be any script thet can be execute by shell and it may call other scripts.
 
.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

       Files: []
       
This clause lets to send arbitrary files to VM instance. The syntax is 

.. raw:: html

   <table><tr><td><pre class="sample">

       Files: [filename1, filename2, ... ]
    
    </pre></td></tr></table>
or

.. raw:: html

   <table><tr><td><pre class="sample">

       Files:
         - filename1
         - filename2
         - ...
    </pre></td></tr></table>
    
    
These files will be transferred on to VM instance. 
If a file specified as filename1, it will be transferred as a text file.
If a file name is surrounded by angle brackets like <filename1>, it will be base64 encoded before it is sent to VM instance.


.. code-block:: yaml
   :linenos:
   :lineno-start: 1   

       Options:
         captureStdout: true
         captureStderr: true

A function which type is ``Type:Application`` has three return parameters:  stdout, stderr и exitCode
``captureStdout: true`` instaructs murano-agent to capture stdout, 
``captureStderr: true`` - stderr

These fields are not mandatory, the default value is true. 
This example does not have the third option ``verifyExitcode: true``.
It is supposed by default.

**How to create Resources/InstallTomcat.template and Resources/scripts/install_tomcat.sh?**

**Resources/scripts/install_tomcat.sh**
The script file whould install Tomcat in a startdard unix way. For ubuntu it would be:

.. raw:: html

   <table><tr><td><pre class="sample">

      #!/bin/bash
      sudo apt-get update
      sudo apt-get -y -q install tomcat7
      sudo iptables -I INPUT 1 -p tcp -m tcp --dport 8080 -j ACCEPT -m comment --comment "by murano, Tomcat"

    </pre></td></tr></table>
    

**Resources/InstallTomcat.template**
is very similar to Restart.template. It is even more simple, because we do not need to pass any parameter to bash script.

.. raw:: html

   <table><tr><td><pre class="sample">

      FormatVersion: 2.0.0
      Version: 1.0.0
      Name: Install Tomcat
      
      
      Body: |
        return deploy().stdout
      
      Scripts:
        deploy:
          Type: Application
          Version: 1.0.0
          EntryPoint: install_tomcat.sh
          Files: []
          Options:
            captureStdout: false
            captureStderr: true
            
      </pre></td></tr></table>
  