---
layout: post
title: "Other People's Code and Ternary Operators"
date:   2015-1-7
categories: ruby
---

I;m working on a project using [jcs/lobsters](https://github.com/jcs/lobsters), a nice Hacker News clone. This is the first time I've ever forked a project and dug into somebody else's codebase. I was a little apprehensive about having to read and understand a bunch of code and was tempted to build something from scratch to avoid it. I'm glad I didn't because it's proving to be an excellent chance to learn some new stuff.

For example, today while I was going through my app's views, hunting down strings to add to my i18n YAML file, I can across this:

{% highlight erb %}
 <% links = {
          "/" => @cur_url == "/" ? Rails.application.name : t('views.layouts.header.home'),
          "/recent" => t('views.layouts.header.recent'),
          "/comments" => t('views.layouts.header.comments')
        } %>
{% endhighlight %}

First off, I didn't recognize [the ternary operator]() and had to refresh my memory.

{% highlight ruby %} boolean_expression ? true_expression : false_expression {% endhighlight %}

This, plus all of those strings stuffed into that hash had me confused for a moment. After looking further down the page, I found this bit of code iterates through the above links hash and creates an  `<a href></a>` for each key and value. 

{% highlight erb %}
#Creates a new link for each pair in the hash.
<span class="headerlinks">
    <% links.each do |u,v| %>
        <a href="<%= u %>" <%= u == @cur_url ? raw("class=\"cur_url\"") : "" %>><%= v %></a>
    <% end %>
</span>
{% endhighlight %}

There's also some other links that get merged into the hash in between but that's enough to get the point. This is equivalent to what I usually do with a bunch of if else statements which check for the existence of a user id in the session or the current url.

{% highlight erb %}
#The way I usually do it.
<% if session[:customer_id] %> 
    <li><%= link_to (t "layouts.account"), customer_path(id: session[:customer_id])  %></li>
    <li><%= link_to (t "layouts.logout"), customer_logout_path, method: :delete %></li>
<% else %>
    <li><%= link_to (t "layouts.login"), customer_login_path %></li>
<% end %>
{% endhighlight %}

This method used in the jcs/lobsters project is slick, and I can see how it saves lines when you need to check a bunch of conditions, but I'm not sure if it's actually better. The ternary operator is less readable for me, especially when jammed into the middle of that hash, and at least a [few people on Reddit seem to agree](http://www.reddit.com/r/ruby/comments/1mcpsm/do_you_use_the_ternary_operator/).

Regardless, I'm glad I decided to fork another project instead of building it myself. I can already tell I'm going to learn a lot from doing it this way. Not to mention I'll be finished with what I was trying to make a lot quicker.
