# Test Kitchen - Vagrant Custom configuration

If you use test kitchen with the kitchen-vagrant plugin and also use VirtualBox then there is some good news, you can add custom configuration.

Change CPU, Memory or Enable clipboard? No problem ... read on.

## Background

When you use test kitchen with the kitchen-vagrant plugin what happens is this
* Test kitchen creates a Vagrant file
* Test kitchen runs Vagrant
* Vagrant takes its instructions from the Vagrant file created by test kitchen and handles all communication with Virtual Box

When test kitchen has created the Vagrant file you are free to view it, the location is normally in
* current folder/.kitchen/kitchen-vagrant/suite name

For a while now Vagrant has supported custom settings for VirtualBox, you can read more here: https://www.vagrantup.com/docs/virtualbox/configuration.html

The custom settings are supplied as key/value pairs and relate to the options for the VirtualBox vboxmanage which are here: https://www.virtualbox.org/manual/ch08.html

Generally this means if you can change a setting with vboxmanage you can change a setting with vagrant using it's customize methods.

The custom settings are exposed in kitchen-vagrant plugin: https://github.com/test-kitchen/kitchen-vagrant#-customize

The .kitchen.yml file in this repo is has an example of custom settings, to test it out pull the repo and run

````
kitchen create default
````

## What if test kitchen doesn't support my setting?

If the test kitchen kitchen-vagrant plugin does not support your Vagrant setting then you can easily use a custom Vagrant file.

To create an example Vagrant file you can simply type Vagrant init on the command line, like to

````
vagrant init
````

Then start editing, the format of the file as used by test kitchen is the same as a Ruby here document, meaning values from test kitchen are passed in and can be accessed in the normal Ruby way, the settings are stored in a variable named config

```ruby
c.vm.box = "<%= config[:box] %>"
```

To use your custom Vagrant file add the following line in your .kitchen.yml file under platforms/driver

````
platforms:
- name: windows2012r2
  driver:
    communicator: winrm
    vagrantfile_erb: VagrantfileWinrm.erb    
````

### Personal customisation

If you have personal customisations which you do not want to share with others then test kitchen allows you to "override" the default settings in the .kitchen.yml file in two ways
* Create a .kitchen.local.yml, this will automatically be pulled in by test kitchen when you use the kitchen commands such as create, converge, test, verify etc.
* Copy your .kitchen.yml to another name such as .kitchen.ci.yml and make changes, create an environment variable called KITCHEN_YAML and set it to your configuration filename, i.e. KITCHEN_YAML=.kitchen.ci.yml

The latter command is useful if you have multiple kitchen configurations such as .kitchen.ec2.yml.

The environment variable also makes access different configurations in Rake file easier or for putting in your ci build configuration file, i.e. appveyor.yml

````
environment:
  machine_user: test_user
  machine_pass: Pass@word1
  machine_port: 5985
  KITCHEN_YAML: .kitchen.ci.yml
````
