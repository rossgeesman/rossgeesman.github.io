---
layout: post
title: "Gems for Asynchronous Tasks"
date:   2014-12-28
categories: gems
---

Delayed Job
-----------
Probably the simplest, quickest way to set up asynchronous tasks in Rails. Delayed job uses Active Record out of the box to store scheduled jobs, which means it requires very little setup and configuration. It does have support for other data stores like Mongoid, and Redis, but I've never used them.

First install as explained at the [Delayed Job Github page][delayed-job]. Then run the following command to create the required table in the database.

{% highlight bash %}
$ rails generate delayed_job:active_record
$ rake db:migrate
{% endhighlight %}


The schema for our new database table should take the following format:

{% highlight ruby %}
create_table :delayed_jobs, :force => true do |table|
  table.integer  :priority, :default => 0      
  table.integer  :attempts, :default => 0      
  table.text     :handler                      
  table.text     :last_error                   
  table.datetime :run_at                       
  table.datetime :locked_at                    
  table.datetime :failed_at                    
  table.string   :locked_by                    
  table.string   :queue                  
  table.timestamps
end
{% endhighlight %}

You can modify Delayed Job's default behavior at config/initializers/delayed_job_config.rb in the following matter: 

{% highlight ruby %}
Delayed::Worker.destroy_failed_jobs = false
Delayed::Worker.max_attempts = 3
Delayed::Worker.max_run_time = 5.minutes
Delayed::Worker.read_ahead = 10
Delayed::Worker.default_queue_name = 'default'
Delayed::Worker.delay_jobs = !Rails.env.test?
Delayed::Worker.raise_signal_exceptions = :term
Delayed::Worker.logger = Logger.new(File.join(Rails.root, 'log', 'delayed_job.log'))
{% endhighlight %}

Queing up a process with delayed task can be accomplished in two ways. The first is super simple. Say for example we have something like a mailer which we don't want to wait for during a user request. If the code for that looks like this normally:

{% highlight ruby %}
def send_confirmation_email
    CustomerMailer.order_confirmation(self)
end
{% endhighlight %}
With Delayed Job, all we have to do is add the .delay method as shown below:
{% highlight ruby %}
def send_confirmation_email
    CustomerMailer.delay.order_confirmation(self)
end
{% endhighlight %}

The other main method of using Delayed Job is through what they refer to as a custom job.

{% highlight ruby %}
MyCustomJob = Struct.new(:foo, :bar) do
  def perform
    #do some stuff using foo and bar you fed to the struct
  end
end
{% endhighlight %}

I put my custom jobs in app/jobs/my_job.rb. The custom job can be called from any place in your app like this.

{% highlight ruby %}
Delayed::Job.enqueue MyCustomJob.new(:foo => something, :bar => something_else)
{% endhighlight %}

To process your queued jobs, use the following rake command from the terminal. 

{% highlight bash %}
$ rake jobs:work
{% endhighlight %}

Resque
------

Resque uses Redis, an in-memory key value store, instead of ActiveRecord for its queue. Installing the Resque gem should install the recommended version of Redis as well.

{% highlight bash %}
$ gem install resque
{% endhighlight %}

Resque tasks follow the form below and you should store them in their own folder such as jobs/myworker.rb or workers/myworker.rb. Each worker will have its own named queue which you specify with the @queue variable.

{% highlight ruby %}
class MyWorker
  @queue = :name_of_queue
  def self.perform(foo, bar)
    #Do something with foo and bar
  end
end
{% endhighlight %}
As with Delayed Job, you can start up a Resque worker from anywhere in your code like this:
{% highlight ruby %}
Resque.enqueue(MyWorker, something.foo, something_else.bar)
{% endhighlight %}

Just remember to run a rake task for each of your queues. This starts up a single worker for each of your queues and work through the jobs in them.

{% highlight bash %}
$ QUEUE=name_of_que rake resque:work 
or
$ COUNT=5 QUEUE=name_of_que rake resque:workers
{% endhighlight %}

One cool feature is Resque's web monitoring tool which allows you to keep tabs on your Resque workers. This can be started up in development using the below command which starts up the Sinatra based app.
{% highlight bash %}
$ resque-web
{% endhighlight %}

Sidekiq
-------

Sidekiq is similar to Resque in that it also uses Redis for the job queue, and you set it up by making your workers in something like app/workers/my_worker.rb as you can see below.
{% highlight ruby %}
class MyWorker
  include Sidekiq::Worker
 
  def perform(foo, bar)
    #Do some stuff with foo and bar.
  end
end
{% endhighlight %}

You can call on your worker from anywhere in your app as in this example:
{% highlight ruby %}
MyWorker.perform_async(foo, bar)
{% endhighlight %}

However, Sidekiq doesn't use rake. Instead you can fire up your workers with the following.

{% highlight bash %}
$ bundle exec sidekiq
{% endhighlight %}

Sidekiq's main difference is that it uses Ruby multithreading to make it more memory efficient than Resque. This requires that you use something like JRuby to get the performance benefit as MRI uses a threadlock.

What Should I Use?
------------------

###Delayed Job
Good:
+ Very Quick and Easy
+ No need for creating additional worker/job folders and files
+ No need to use Redis
Bad:
-Usage of ActiveRecord for queue means a you take a performance hit
-No fancy web dashboard

###Resque
Good:
-More performative due to Redis instead of ActiveRecord
-Nice web dashboard for monitoring processes
Bad:
-More setup to worry about

###Sidekiq
Good:
-Can be very fast and memory efficient when using multithreaded Ruby
-Has a dashboard
-Can use it in combination with Resque as it uses the same architecture
Bad:
-Need to be concerned with threadsafety of code and libraries
-More setup and configuration than DelayedJob




[delayed-job]: https://github.com/collectiveidea/delayed_job