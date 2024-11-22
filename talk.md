# Talk

## Intro


## Puppet overview

I appreciate that there may be people in the room that haven't used Puppet before and some that done even know what Puppet is so I'll give you a brief overview.

### Resources

Puppet manages resources, a resource can be a file, a user, a package. It's a thing on your machine that you want to manage and we do that defining it's desired state and we write that desired state with Puppet DSL, Domain Specific Language.

Here's a resource.

```puppet
service { 'ntp' :
  enable => true,
  ensure => running,
  }
```

I've heard many people say Puppet is a difficult language to learn, it's not, this is a perfect example, we want the ntp service running and we want it enabled when our machine boots up. It's so simple we don't even need to comment our code.

We can collect like resources together into a class.

```puppet
class { ntp
  package { 'ntp' :
    ensure => present,
    }

  file { /etc/ntp/.conf :
    ensure => file,
    contents => "server time.google.com",
    }

  service { 'ntp' :
    enable => true,
    ensure => running,
    }
}
```
  
* This should be pretty self explanatory we want to install the ntp package.
* We want to configure it's config file, we can use templates or static files, I just want to keep this simple so we just going put a single time source into it.
* As we've already seen, we want to start the service.

### Ordering

A little on ordering in Puppet.

If like me, you've come from a sysadmin background, you've probably spent a long time writing scripts in things like bash and so looking at the code you'll be thinking, that's all in the right order, shouldn't be any problems there. That is not the case, Puppet will read the file from top to bottom but by the resources are treated as individual components, we don't know what order the agent will apply them in. So we use ordering meta parameters.

```puppet
class { ntp
  package { 'ntp' :
    ensure => present,
    }

  file { /etc/ntp/.conf :
    ensure => file,
    contents => "server time.google.com",
    require  => Package["ntp"],
    }

  service { 'ntp' :
    enable => true,
    ensure => running,
    subscribe => File["/etc/ntp.conf"],
    }
}
```

* The file resource requires the package resource to have been applied.
* The service resource requires the file resource to have been applied but also "watches" the file resource and will restart the service will restart if that resource changes in any way.

### Modules

Our class would typically be part of a Puppet module, a module contains a class or classes all related to the thing the module does. It can also contain data, static files, template files and dependency information so everything can exist in one place and these can then exist as a repo on Gitlab, Github BitBucket etc. 

### Roles and Profiles

You can assign a module directly to a node, but we probably dont want to just manage ntp on our node, what would be really useful would be to group ntp and maybe a few other modules like ssh for example, or sudo, into baseline. We can do that with a Profile.

```puppet
class profile::baseline {
  include ntp
  include ssh
  include sudo
}
```

The profile is just another module, there is nothing special about it, it's just a convention where we use another module as an abstraction layer. I could create another profile.

```puppet
class profile::apache {
  include apache
}
```

And I could keep doing this and build up a library of profiles for everything I want to manage. So when someone says "I need an apache webserver for the marketing department" you can say, "aha, a new role...".

A role is another abstraction layer, just like the profile.

```puppet
class role::marketing_webserver {
  include profile::baseline
  include profile::apache
}
```

This is where we can see the real power of Puppet as an automation tool, we can respond to customers requests for new server, not in days, but in minutes, we just assemble a set of prebuilt components.

But if I've got to write a module for everything, that's going to take forever... you don't. We have the Puppet forge [Puppet Forge](https://forge.puppet.com/), there are over 7,500 modules on the Forge and everything in the profiles I've just shown you is on there. 
Some are owned by Puppet
Some are community maintained

If you want to build your own modules check out the [PDK](https://www.puppet.com/docs/pdk/3.x/pdk.html) it has everything you need to create and test a module.


## What not to do

Use Puppet to deploy a bash script.

Hier.yaml 



## How to fix them
