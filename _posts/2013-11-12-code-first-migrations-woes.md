---
layout: post
title: "Unable to generate an explicit migration/Unable to update database"
permalink: /code-first-migrations-woes/
---
While working on a small feature for something at work today I needed to add a field in to one of our domain entities. It really was a tiny change:
{% highlight csharp %}
public DateTime? LastUpdated { get; private set; }
{% endhighlight %}

After continuing on to add unit tests etc I tried to add the migration:

{% highlight text %}
Add-Migration EnabledProjectLastUpdated
{% endhighlight %}

This threw a big red error in my face:
{% highlight text %}
Unable to generate an explicit migration because the following explicit migrations are pending: [201311010453495_InitialProduction]. Apply the pending explicit migrations before attempting to generate a new explicit migration.
{% endhighlight %}
So I did the logical thing and tried to re build my database locally: 
{% highlight text %}
Update-Database
{% endhighlight %}

To be greeted by another error:
{% highlight text %}
Unable to update database to match the current model because there are pending changes and automatic migration is disabled. Either write the pending model changes to a code-based migration or enable automatic migration. Set DbMigrationsConfiguration.AutomaticMigrationsEnabled to true to enable automatic migration.
You can use the Add-Migration command to write the pending model changes to a code-based migration.
{% endhighlight %}

After going around in circles dropping my localDB, re-attemtping the update and re-attempting the add  for rather longer than I'd like to admit I just tried a simple 
{% highlight text %}
Add-Migration
{% endhighlight %}
For some reason this enabled me to add the Migration when I specified the name and run 
{% highlight text %} update-database {% endhighlight %} 
as if nothing had happened. I thought I'd write it down to help anyone else (or myself) if they ran into this.