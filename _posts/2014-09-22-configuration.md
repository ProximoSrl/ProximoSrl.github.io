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

- When a software is composed by multiple services it is good to have a single configuration
- Config file mixes application specific settings with .NET settings (es. assembly redirect, )
- Xml is not the best human readable format
- Frictions for automatic deploy ([web.config transform](http://msdn.microsoft.com/en-us/library/vstudio/dd465318(v=vs.100).aspx), [SlowCheetah](http://visualstudiogallery.msdn.microsoft.com/69023d00-a4f9-4a34-a6cd-7e854ba318b5), etc)

###Resist the urge for complexity
As a good rule of thumb, avoid creating an architecture capable of launching rocket into space if you only need a better configuration manager for a set of services. 

> 
Simplicity is the ultimate sophistication. [Leonardo da Vinci](http://www.brainyquote.com/quotes/quotes/l/leonardoda107812.html)
>
>Keep It Simple, Stupid. [Wikipedia](http://en.wikipedia.org/wiki/KISS_principle)

One of my defects is loving software architecture and this usually lead to an unnecessary complexity. In latests period I'm trying to change attitude, the goal is doing the simplest thing that could possibly work. This is what I think:

> 
Solving simple problem with complex solution, #FAIL.
>
Solving simple problem with simple solution, #GOOD.
>
Solving complex problem with complex solution, #GOOD.
>
Solving complex problem with simple solution, #GENIUS.

Our need "now" is moving settings away from multiple config file and having a single "configuration manager"; json file with all the configuration is good enough. Then we need to address some pains in configuration, this leads to a simple Configuration Manager class that wraps access to json configuration file with some facilities.

###Keep in mind DevOps 
Releasing your sofware [must be easy and well documented procedure](http://devopsreactions.tumblr.com/post/57234308379/setting-up-a-product-following-vendors-instructions) and you should help as much as possible Operational people in troubleshooting wrong configruations.

One of the pain point we want to address with the new configuration manager is getting rid of this code.

{% highlight csharp %}
var sysUrl = new MongoUrl(ConfigurationManager.ConnectionStrings["system"].ConnectionString);
var sysdb = new MongoClient(sysUrl).GetServer().GetDatabase(sysUrl.DatabaseName);
{% endhighlight %}
	
This code can bite you badly. In case of missing connection string) when the service starts it chrashes without any useful log or clue of what is wrong. Logging raw exception can be good for developers, but fails to be useful for operational people. If you got a MongoException or a NullReferenceException with a stack trace you have absolutely no clue of **what is wrong** and **where you can fix the error**. One of the goal of good configuration manager is making simple to give useful information in case of wrong setting.

Thanks to a new configuration manager, I can simply substitute standard ConfigurationManager with the new implementation, leading to code version 2.

{% highlight csharp %}
var sysUrl = new MongoUrl(CqrsConfigurationManager.Instance.GetSetting("connectionStrings.system"));
var sysdb = new MongoClient(sysUrl).GetServer().GetDatabase(sysUrl.DatabaseName);
{% endhighlight %}

The only difference is in the new CqrsConfigurationManager class; if the setting is missing I got a meaningful log: *Required setting 'connectionStrings.system' not found in configuration: http://localhost/Jarvis/Debug/config.json*. This log gives two important information: **What** is wrong and **where** I should add missing configuration.

###Fail as first as possible (and with good log)

The  situation is still not optimal, *what happens if the configuration is present but is wrong (es. wrong mongo connection string)*? The answer is: **it depends**. Usually the code will throw ***some*** exception ***somewhere*** and only ***when*** the configuration is really used. The ***when*** problem is one of the worst. Suppose your software needs the address of a special mongo instance where it should save commands that raised exception during execution. If that settings is wrong the service starts without any error in the logs, giving you false confidence that everything is good.

After a while log contains error with MongoDbException: *Unable to connect to server fingolfin:27017: No such host is known.* and absolutely no clue on how to fix it. 

The rule is: ***I want my software to detect as soon as possible wrong configuration settings, and log every information needed to do the fix!***. This lead  to code version 3.

{% highlight csharp %}
MongoDatabase sysdb = null;
CqrsConfigurationManager.Instance.GetSetting("connectionStrings.system", setting =>
{
    var sysUrl = new MongoUrl(setting);
    sysdb = new MongoClient(sysUrl).GetServer().GetDatabase(sysUrl.DatabaseName);
    sysdb.GetStats();
});
{% endhighlight %} 

The semantic is simple, GetSetting accepts a lambda that should **consume and validate the setting in a single step**. If no exception is thrown or lambda returns empty string, configuration is considered to be good. If connection string is wrong I immediately got an exception: *Error during usage of configuration 'connectionStrings.system' - Config location: http://localhost/Jarvis/Debug/config.json - error: Unable to connect to server fingolfin:27017: No such host is known.*. This log contains enough details to understand **what** configuration is wrong **where** it can be fixed.

This is especially good if the setting is complex, ex:a simple comma separated list of integers

{% highlight csharp %}
List<Int32> parsedConfig = new List<int>();
CqrsConfigurationManager.Instance.GetSetting("coefficientList", setting =>
{
    var splitted = setting.Split(',');
    Int32 tmp;
    foreach (var number in splitted)
    {
        if (!Int32.TryParse(number, out tmp))
        {
            return "The value " + number +
                    " is not a number. I'm expecting a list of integer number comma separated.";
        }
        parsedConfig.Add(tmp);
    }
    return "";
});
{% endhighlight %}

If a wrong string *13,24,3O9* is found in configuration file (there is an O letter instead of zero in last number), the system will log this message: ***Error during usage of configuration 'coefficientList' - Config location: http://localhost/Jarvis/Debug/config.json - error: The value 3O9 is not a number. I'm expecting a comma separated list of integer.***. As a general rule you can adopt a little variant of one of the rule of [Code for the maintainer](http://c2.com/cgi/wiki?CodeForTheMaintainer) 

>
Always code as if the person who ends up installing your software is a violent psychopath who knows where you live! 

Next post will deal on: *Where do you specify location of config file?* 


