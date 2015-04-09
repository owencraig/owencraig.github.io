---
layout: post
title: "CS0135: 'Model' conflicts with the declaration 'System.Web.Mvc.WebViewPage<TModel>.Model'"
permalink: /cs0135-model-conflicts-with-the-declaration-system-web-mvc-webviewpagetmodel-model/
comments: true
---
After adding a declaration to pipe in some text from our model in a view in one of our MVC apps:
{% highlight csharp %}
@Model.Project
{% endhighlight %}

I was hit with the above error message:
{% highlight text %}
CS0135: 'Model' conflicts with the declaration 'System.Web.Mvc.WebViewPage<TModel>.Model'
{% endhighlight %} it turns out that the person that had originally authored the view had used the upper case Model in some HtmlHelper calls like so:

{% highlight csharp %}
@Html.HiddenFor(Model => Model.Project)
{% endhighlight %}
A quick change to using either the lower case variant of model (or even a completely different identifier) for the parameter of the lambda fixed this up. 

Hopefully this helps someone else when they come across it.