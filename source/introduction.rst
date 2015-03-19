Introduction to Murano
======================
 
What I can do with Murano
-------------------------

Murano is an application catalog. 
It provides user-friendly interface and gives you opportunity to interact with different cloud-ready applications.

Murano has a UI plugin for OPenstack Horizon dashbord called Murano dashboard.
This is how Murano web interface looks like:

.. todo:: insert image here

This is the catalog of all applications available in your tenant.
Each application has a name, short description, detailed information.
Let's take a look at the Tomcat application:

.. todo:: insert image here

When you deploy it, it will create a new instance, configure its security group rules, and install Tomcat on it. 


You can create your own Murano application, package it as zip archive and upload to Murano.
The source code of your application packaged as zip archive is called package. 
Once uploaded,  your application is immediately availale in the application catalog. (see the screenshot #1) 

Package management is available in definitions panel.
This is the interface for application publisher and catalog administrator

.. todo:: insert image here


This interface allows you to download any package as well. 
This is a very useful if you are new to Murano. 
You may look how every application is done and create your own application based on the complex application that is already present in the catalog.

Environment panel.

.. todo:: insert image here


You use this panel when you need to deploy an application.
Environmant is a set of applications managed by a single tenant.
Appplications within single environmanet may comprise complex configuration, 
while applications in different environmetns are always independent from each other.
Environment is a logical agregation of multiple applications that binds them together.

Let's say that we need to deploy for QA two applications logically dependent on each other: 
Tomcat that hosts web application, ans MsSQL databes, where hosted web application retrieves data from.

Let's create a new environment (env-petstore-qa),

.. todo:: insert image here

 
All both appplications to the env

.. todo:: insert image here


And lets deploy the environment

.. todo:: insert image here


After deployment is completed, we can see that both instances are created 
on Murano topology page

.. todo:: insert image here


and on instances page

.. todo:: insert image here


Imagine, that besides QA team, you have a group of users who perform User acceptance tests on your application.
They need their own Tomcat and Database to test on.
Let's create env-petstore-uat for them and deploy their own Tomcat and MySql:  

.. todo:: insert image here


Murano vs Heat
--------------
Murano translates the source code of a package into Heat Orchestration template.
It is Heat Orchestartions template that is executed when you deploy Murano application.

When should I prefer Murano over Heat? 

Firstly, if you have little experience with Openstack, you may profit from the fact that 
you do not have to know which particular Heat template do you need.
Murano offers application search, which includes search by category (aka tag).
These are: Web, Databases, Application Servers, Big Data, Load Balances, etc. just to name a few.
Each application is a tested piece logic that not only installs required software component, but also performs all the 
implied instance configuration (such as to ensure that you will be to access HTTP port 8080 if you provisioned Tomcat instance)
All these security configurations are transparent to you.

Secondly, Murano provides actions.
They allow you to modify deployment configuration after application is deployed.
Read more on actions here: http://openstack.10931.n7.nabble.com/Glance-Heat-Murano-split-dsicussion-td49855.html
and here: https://www.mirantis.com/blog/seeing-murano-actions-action/                                                                                                                                      

And lastly, as a Software Engineer you would favor Murano over Heat templates 
for any  progranning task where you would favor imperative programming language over declarative programming language.
Imperative program contains list of instructions. They are executed one by one from the first to the last and produce desired result.
Imperative programming languages are Java, C,... and MuranoPL. 

Declarative program describes what should be the results, but does not specify the order of execution.
SQL "SELECT FROM WHERE <boolean condition>" query, puppet manifest,... and Heat template are declarative programs. 
The weak point of a declarative program is a dependency management. You cannot just write "give me an instance and rpm  installed".
You need to specify that you install rpm only after the instance is created. I.e. that rpm installation "depends on" instance creation.
It is not a big deal for a small program, but quickly becomes a daunting task as you program increases in size. 

For more details see section "Murano vs Heat Extensive example"


What is inside Murano package
-----------------------------

The structure of a Murano package is:

\<code\>
   ..
   |_  manifest.yaml
   |
   |_  Classes
   |   |_  MyApplication.yaml
   |
   |_  Resources
   |   |_  DeployMyApplication.template
   |   |_  scripts
   |       |_installer.sh
   |       |_setenv.sh 
   |
   |_  UI
   |   |_  ui.yaml
   |
   |_  logo.png
   
\</code\>
   
:manifest.yaml is an entry point to the package. It is in yaml format and contains the general information
about the appplication such as name, author and description

:Classes folder contains Murano templates written in MuranoPL language. They
define properties and methods of application components.

Class methods contain references to the application plans, which control installation process on a virtual machine.
:Resources folder contains these application plans.

:Resources/scripts folder contain  executable scenarios that are used by execution plans.

:UI folder contain a description of the UI form for your application. UI form will be rendered into html popup window, 
where you will be abple to pass parameters for your application.

:logo.png in a logo of your application.  It is displayed on Muarno dashboard. The file is recommended, but not required.

This folder structure must be packaged into zip archive <MyApplication>.zip

We will get into more details of what is inside *.yaml and *.template files in the next chapter: Simple VM Application.
 

Simple VM Application
---------------------
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
