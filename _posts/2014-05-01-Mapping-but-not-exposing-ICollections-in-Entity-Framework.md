---
layout: post
title: Mapping but not exposing ICollections in Entity Framework
permalink: /mapping-but-not-exposing-icollections-in-entity-framework/
---
Jimmy Bogard [wrote a post](http://lostechies.com/jimmybogard/2014/04/29/domain-modeling-with-entity-framework-scorecard/) a couple of days ago sizing up EF 6 for fitness in use for a fully encapsulated domain model.

*UPDATE: As people pointed out I made a mistake when I first wrote this, the properties should be protected internal, not private internal (since that makes no sense)*

One of the points he brought up was the lack of ability to encapsulate collections. After I wrote a comment alluding to a way that my team and I use to (kind of) work around this, his asked for more information which kicked me into gear to write this post.

### The problem
If we're writing truly encapsulated domain models we should be creating classes like so:
{% highlight csharp %}
public class Parent
{
    private readonly ICollection<Child> _children;
    
    public IEnumerable<Child> Children
    {
    	get
        {
        	return _children.Skip(0);
        }
    }
    
    public void AddChild(Child child)
    {
    	// some domain logic to ensure internal consistency
        _children.Add(child);
    }
    
    // some other methods that mutate state
}
{% endhighlight %}
As an aside, the .Skip(0) is a small hack just so that the client isn't as easily able to cast back to an ICollection and shoot themselves in the foot.

The problem with this is that in order for Entity Framework to map this collection, it has to be able to *get* to it. If our context is in the same assembly as our models, we can make the collection internal but I often try to abstract my db and infrastructure code out to its own assembly. As it currently stands (in this instance), EF is only able to map to public Properties, meaning our class ends up being (at best):
{% highlight csharp %}
public class Parent
{
    public ICollection<Child> Children { get; private set; }

    // etc etc
}
{% endhighlight %}

You may think this solves the problem because we can't set Children ourselves. This is incorrect, in our client code we're still able to (even accidentally):
{% highlight csharp %}
	Parent parent; 
    Child child;
    // some kind of assignment here
    p.Children.Add(child);
{% endhighlight %}
Oops! We've just walked around our own carefully designed domain methods.


### The solution
In order to let Entity Framework map our private member, we need to expose something that our DbContext can map to. So from our previous example, if we were to refactor our entity class to make the ICollection a private property instead of a read only field:
{% highlight csharp %}
	protected virtual ICollection<Child> ChildrenStorage { get; set; }
{% endhighlight %}
Then we add a static accessor like so:
{% highlight csharp %}
	public static Expression<Func<Parent, ICollection<Child>>> ChildrenAccessor = f => f.ChildrenStorage;
{% endhighlight %}
Then in your EntityTypeConfiguration (or at least in your OnModelCreating within your context [which is what I'm doing here]) we map the relationship like so:
{% highlight csharp %}
	modelBuilder.Entity<Parent>()
    	.HasMany(Parent.ChildrenAccessor)
        .WithRequired();
{% endhighlight %}
This allows Entity Framework to map the collection, whithout exposing the collection directly to any clients. The client is still able to access the collection and work around any of your methods, but only if they go out of their way to do it. In other words you can't completely protect the user (or yourself) from doing the wrong thing, but you can make it so you don't do it accidentally.

### Putting it all together
{% highlight csharp %}
public class Parent
{
	protected virtual ICollection<Child> ChildrenStorage { get; set; }
    
    // accessor for ef to be able to map what we're after
  	public static Expression<Func<Parent, ICollection<Child>>> ChildrenAccessor = f => f.ChildrenStorage;
    
    public IEnumerable<Child> Children
    {
    	get
        {
        	return ChildrenStorage.Skip(0);
        }
    }
    
    public void AddChild(Child child)
    {
    	// some domain logic to ensure internal consistency
        ChildrenStorage.Add(child);
    }
    
    // some other methods that mutate state
}

public class Child
{
	// wonderful domain driven stuff
}

public class ExampleContext:DbContext
{
	public DbSet<Parent> Parents { get; set; }
    public DbSet<Child> Children { get; set; }
    
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
    	modelBuilder.Entity<Parent>()
        	.HasMany(Parent.ChildrenAccessor)
        	.WithRequired();
    }
}
{% endhighlight %}

As I was saying earlier this isn't a foolproof way around the problem, but it does allow us to protect ourselves (and anyone writing client code for our models) from a simple error.