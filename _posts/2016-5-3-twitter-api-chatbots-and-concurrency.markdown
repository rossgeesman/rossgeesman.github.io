---
layout: post
title: "Twitter API, Motivational Chatbots, and Concurrency"
date:   2016-5-2
categories: "concurrency, twitter-api, chatbots"
---

This post is really just notes on the beginning of a little project I decided to work on. Recently I've been having trouble getting motivated to do any non-work projects so I decided to try using a goal tracking app in an attempt to be more productive. In the past, I've tried setting daily or weekly calendar events to remind me to work on things I want to get done, but it's rarely been successful. I looked around at a few options and most of the apps didn't really provide any significantly different features than a calendar. They allow you to set a goal, then check in at predetermined intervals to see if you are following through. The problem with both is that, aside from the intrinsic benefit of accomplishing your goal, there's really no positive benefit to ticking off boxes on a calendar. Likewise, there's no negative feedback that happens when you ignore the daily prompt.

So, I started thinking, why not raise the stakes and make my progress public? Since [chatbots seem to be all the rage][chatbot-article], I decided to make a simple bot that will tweet me daily to check in on the progress of my goals, and keep track of how many consecutive days of progress I've made. Maybe having a Twitter bot call me out in public will serve as enough motivation to get do something extra-curricular?

Ideally I'd like to be able to set goals and ask about my progress all within Twitter so I started out by looking at the [sferik's Twitter gem][twitter-gem] which supports Twitter's Streaming API out of the box. Authorizing is easy enough:

{% highlight ruby %}

  require 'twitter'

  client = Twitter::Streaming::Client.new do |config|
    config.consumer_key        = 'CONSUMER_KEY'
    config.consumer_secret     = 'CONSUMER_SECRET'
    config.access_token        = 'ACCESS_TOKEN'
    config.access_token_secret = 'TOKEN_SECRET'
  end

{% endhighlight %}

Next, I wanted to figure out how to open a connection with Twitter that would feed me any mentions of my bot. There doesn't seem to be a `Twitter::Mention` object type that I can filter for, so using `Twitter::Streaming::Client#filter` method to track mentions of my robot's name seems like the best solution.

{% highlight ruby %}

  client.filter(track: 'goalchicken') do |object|
    puts "#{object.user.user_name} #{object.text}" if object.is_a?(Twitter::Tweet)
  end

{% endhighlight %}

Since I spent too long deciding on a name for my bot (it's [@goalchicken][goalchicken] if you're interested), this is all I had time for in the hour or two I set aside to work on this.

I'm aware I could probably just poll the Twitter REST API at regular intervals to get mentions but what fun would that be? One of my goals with this project is to learn something new, so why not use the opportunity to play around with concurrency? Actually, figuring out how I'm going to handle concurrency is next on my list of tasks. I've done some reading on the major concurrency libraries in Ruby and it seems [EventMachine][eventmachine], [concurrent-ruby][concurrent-ruby], and [Celluloid][celluloid] are the most commonly used. I've heard bad things about [EventMachine][eventmachine] so I'll probably end up using one of the other two.




[chatbot-article]: http://venturebeat.com/2016/05/01/the-200-billion-dollar-chatbot-disruption/
[goalchicken]: https://twitter.com/goalchicken
[twitter-gem]: https://github.com/sferik/twitter
[eventmachine]: https://github.com/eventmachine/eventmachine
[concurrent-ruby]: https://github.com/ruby-concurrency/concurrent-ruby
[celluloid]: https://celluloid.io/