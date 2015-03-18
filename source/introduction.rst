Introduction to Murano
======================
 
What I can do with Murano
-------------------------
Give short overview and define key entities:  Application, Environment, Package

Sample deploy of a single application

Sample deploy of dependent applications
 
      Tomcat and Database? - demonstrate with UI screenshots

Provide Use Case for several environments
      QA vs UAT, etc.
      HR app vs Finance appli


Murano vs Heat
--------------


What is inside Murano package
-----------------------------
   Structure of a package
   -Manifest
   -UI form
   -Template
   -Scripts

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
