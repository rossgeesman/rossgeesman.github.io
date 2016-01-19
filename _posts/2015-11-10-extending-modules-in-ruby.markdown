---
layout: post
title: "Extending Modules and Self"
date:   2015-11-10
categories: "ruby, metaprogramming"
---

Lately I've been reading the book [Metaprogramming Ruby][metaprogramming] and in an early chapter, it talks about Ruby's object model and how modules can be mixed into Classes using `extend Module`. I've read explanations of how Ruby objects work here and there multiple times but seeing it put into practice in the book's example was one of those "oh shit, that's what they were talking about" moments.

I've found that the saying "If all you have is a hammer, everything looks like a nail" definiteley applies to my learning of new programming concepts. After reading the chapter, I started looking for nails and pretty soon, I found one.

At [Fishisfast][ff], we use state machines to model the process that the thousands of shipments go through in our warehouse daily. For example, an incoming package will need to get unpacked, photographed, and stored and each of these steps will have a corresponding state. Since our warehouse workers are really busy it's easy for them to overlook something, resulting in an object remaining in a particular state for too long. This is bad so we decided to make something to alert managers via Slack when something is taking longer than expected.

I decided to make a cron job that would periodically go through and check for objects that have been stalled in one state for too long then fire off messages to the relevant staff. Since we need to do this across multiple objects with multiple states and at different time scales, I figured it would be best avoid the quick and dirty route and make a small, reusable module.

{% highlight ruby %}
module ModuleName
  def foo
  end
end
{% endhighlight %}

This can now be mixed in to any of our model classes arbitrarily so they inherit the Module's methods.

{% highlight ruby %}
class ModelName < ActiveRecord::Base
  extend ModuleName
end

ModuleName.foo
{% endhighlight %}

The problem I ran into is that although there are a lot of standard states across the different models, sometimes they don't always have the same states so the logic for determining 'lateness' is different in some cases.

Since there was only one model that needed to be handled differently, I decided to add code which would check the class for cases that needed to be handled differently.

{% highlight ruby %}
module ModuleName
  def foo
    if self.kind_of? SpecialModel
      # do something
    else
      # do something else
    end
  end
end
{% endhighlight %}

To my surprise this didn't work. `SpecialModel` was getting treated no differntly than any other model. I didn't understand why at first because the code also had ActiveRecord queries like `self.where(foo: 'bar')` and they were working just fine. Shouldn't `self` in this context be SpecialModel?

After thinking about it for longer than I'd like to admit, I realized that I was both right and wrong (at least that's what I told myself to feel better). At runtime `self` IS the class `SpecialModel` itself not an instance of `SpecialModel`. In Ruby everything is an object and classes are no different. `SpecialModel`is just an instance of Class. I verified quickly with `self.class`, getting `Class` back as the answer. I changed the logic in the module to the following and everything worked as expected. 

{% highlight ruby %}
module ModuleName
  def foo
    if self == SpecialModel
      # do something
    else
      # do something else
    end
  end
end
{% endhighlight %}

I'm still not sure if this is the best way of implementing this reminder system but I guess we'll see how it holds up over time. I'm going to read the rest of the book and learn a new trick before I start mixing modules into everything. 


[metaprogramming]: https://pragprog.com/book/ppmetr2/metaprogramming-ruby-2
[ff]: https://fishisfast.com
