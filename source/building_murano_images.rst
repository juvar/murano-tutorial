Building Murano Images
======================

Flavors and images
------------------

When you deploy Murano application, it usually asks to select a flavor and an image. 
(Otherwise flavor and image are somehow selected by the application itself behind the scenes)
It is your responsibility as an application deployer, to select proper values for flavor and image.

Flavor is a size of a virtual machine that will be created. Flavor specifies CPU count, RAM size, storage size.
These parameters should correspond to the requirements of the operating system that resides on the selected image,
as well as to the requirements of all other software that will be installed on the instance.


Murano-enabled image
--------------------

Every instance that is created and managed by Murano always has murano-agent installed on it.
Murano-agent listens to RabbitMQ queue where it receives Murano commands from.

The standard and the most reliable way to have murano-agent installed on the instance, 
is to pre-install murano-agent on the image of the operating system.

The instruction below will describe every step of how to create a murano-enabled image.
It assumes that you have a base image as iso file, and already installed OpenStack.
Your OpenStack must have Image Server 'Glance' and  Murano installed.

The idea of the instruction is to:
1. Register the base image in Glance. Please note that the official distribution does not work: the image must be prepped for OpenStack.
2. Launch an instance from the base image. Let's call it 'base istance'.
3. SSH to base instance and install murano-agent on it from command line. The commands that install murano-agent will be tested on Ubuntu 14.04.
4. Take an image snapshot of the base instance. Let's call it murano-enabled image.
5. Use Murano to add Murano metadata to the murano-enabled image. Metadata not alter an image, it just hangs "Murano" tag o it.

 
 How to create Murano-enabled image
 ----------------------------------
 
 Register the base image in Glance
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 You may skip this step if you have the base image already in Glance.
 
 Launch base instance from the base image
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
 Install murano-agent on base instance
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 ssh -i <path to your private key> ubuntu@<floating ip of base instance>


.. todo:: Steps below does not work. We need to fix them


 
   sudo apt-get update
   sudo apt-get install git-core wget make gcc python-pip python-dev python-setuptools python-virtualenv
   sudo mkdir -p  /opt/stack/murano-agent
   sudo git clone --depth=1 -b master git://git.openstack.org/stackforge/murano-agent.git /opt/stack/murano-agent
   sudo mkdir -p /etc/murano
   
      
sudo vi /etc/murano/agent.conf


[DEFAULT]
debug=True
verbose=True
log_file = /var/log/murano-agent.log
storage=/var/murano/plans

[rabbitmq]

# Input queue name
input_queue = agent-tasks

# Output routing key (usually queue name)
result_routing_key = agent-results

# Connection parameters to RabbitMQ service

# Hostname or IP address where RabbitMQ is located.
host = localhost

# RabbitMQ port (5672 is a default)
port = 5672

# Use SSL for RabbitMQ connections (True or False)
ssl = False

# Path to SSL CA certificate or empty to allow self signed server certificate
ca_certs =

# RabbitMQ credentials. Fresh RabbitMQ installation has "guest" account with "guest" password.
login = guest
password = guest

# RabbitMQ virtual host (vhost). Fresh RabbitMQ installation has "/" vhost preconfigured.
virtual_host = /
   


   sudo install -D -g root -o root -m 0755  /opt/stack/murano-agent/contrib/elements/murano-agent/install.d/murano-agent.conf /etc/init
   sudo install -D -g root -o root -m 0755  /opt/stack/murano-agent/contrib/elements/murano-agent/install.d/murano-agent.service /etc/systemd/system/
   
 
 
 Take an image snapshot from base instance where murano-agent is installed
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
 Register an image in Murano
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~