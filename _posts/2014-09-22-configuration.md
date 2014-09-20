---
layout: post
title: Configurations
description: "Jarvis configuration as a service"
tags: [jarvis]
author: gianmariaricci
image:
  feature: gears-configuration.jpg
  credit: wonderfulengineering.com
  creditlink: http://wonderfulengineering.com/32-amazing-gear-wallpaper-backgrounds-in-hd-for-download/
comments: true
share: true
---

## Why configurations are important
When you start working with .NET technologies it is natural to place application settings inside app.config/web.config, but this approach usually fails to be useful in long term for many reasons

- When a software is composed by multiple services, it is useful to have one common configuration file for all services that belongs to the same "suite"
- .NET config file mix application settings with .NET specific settings (es. assembly redirect)
- Xml is not the best human readable format
- Frictions for automatic deploy ([web.config transform](http://msdn.microsoft.com/en-us/library/vstudio/dd465318(v=vs.100).aspx), [SlowCheetah](http://visualstudiogallery.msdn.microsoft.com/69023d00-a4f9-4a34-a6cd-7e854ba318b5), etc)

###Resist the urge for complexity
You should resist the temptation to design an archtiecture for launching a rocket into space if you need only a better configuration manager configuration for a set of services. 

> 
Simplicity is the ultimate sophistication. [Leonardo da Vinci](http://www.brainyquote.com/quotes/quotes/l/leonardoda107812.html)
>
>Keep It Simple, Stupid. [Wikipedia](http://en.wikipedia.org/wiki/KISS_principle)

One of my greatest defects it loving architecting and this lead quite often to an unnecessary complexity, so I'm trying to change my attitude.

> 
Solving simple problem with complex solution, #FAIL.
>
Solving simple problem with simple solution, #GOOD.
>
Solving complex problem with complex solution, #ACCEPTABLE.
>
Solving complex problem with simple solution, #GENIUS.

In my first implementation, a json file with all the configuration is good enough, it centralize configuration and I can add some functionalities that helps handling configurations.

###Keep in mind DevOps 
Releasing your sofware [must be easy and well documented procedure](http://devopsreactions.tumblr.com/post/57234308379/setting-up-a-product-following-vendors-instructions) and you should help as much as possible Operation people in understadingwhat is wrong.

One of the pain we want to easy changing how configuration work is getting rid of this kind of pieces of code.

{% highlight csharp %}
var sysUrl = new MongoUrl(ConfigurationManager.ConnectionStrings["system"].ConnectionString);
var sysdb = new MongoClient(sysUrl).GetServer().GetDatabase(sysUrl.DatabaseName);
{% endhighlight %}
	
In this simple innocent two lines we have some bad problem that can bite you during deployment. If you remove a connection string from config file, or you forget to add a new connection string after a new deploy, you just run the service and it chrashes without any log and any useful information. One of the goal of a good configuration system is clearly telling to the user what configuration is wrong.

If a connection string or other configuration is missing I want to know what is wrong, so this is configuration code version 2.

{% highlight csharp %}
var sysUrl = new MongoUrl(CqrsConfigurationManager.Instance.GetSetting("connectionStrings.system"));
var sysdb = new MongoClient(sysUrl).GetServer().GetDatabase(sysUrl.DatabaseName);
{% endhighlight %}

The only difference is in the new CwrsConfigurationManager class; now if required setting was not found I got this error logged with log4net (both udp and file): *Required setting connectionStrings.system not found in configuration: http://localhost/Jarvis/Debug/config.json*. Such an error immediately give me two indication: What is wrong and where I should fix the configuration.

###Fail as first as possible (and with good log)

The previous situation is still not optimal, *what happens if the configuration is present in configuration manager but is wrong (es. wrong mongo connection string, bad configuration)*? The answer is: **it depends**. Usually the code will throw ***some*** exception ***somewhere*** and hopefully you got a log, but quite often the log is not useful.

A log telling you that you got a MongoDbException and the message is *Unable to connect to server fingolfin:27017: No such host is known.* and a nice stack trace is not so useful for Ops or for anyone trying to setup your software.

***I want my software to fail as soon as a wrong configuration setting is found, giving much information as possible to the user***
{% highlight csharp %}
MongoDatabase sysdb = null;
CqrsConfigurationManager.Instance.GetSetting("connectionStrings.system", setting =>
{
    var sysUrl = new MongoUrl(setting);
    sysdb = new MongoClient(sysUrl).GetServer().GetDatabase(sysUrl.DatabaseName);
    sysdb.GetStats();
});
{% endhighlight %}

The semantic is simple, I can pass a simple lambda that uses the setting, and if no exception is thrown configuration is considered to be good. With the previous code, if the Mongo Server does not respond or if the settings is wrong I got an exception: *Error during usage of configuration 'connectionStrings.system' - Config location: http://localhost/Jarvis/Debug/config.json - error: Unable to connect to server fingolfin:27017: No such host is known.* that contains enough details to immediately find and fix what is wrong.

Next post will deal on: *Where do you specify location file?* 


