---
layout: post
title: "Moving Intercom Conversations Between Instances"
date:   2016-1-18
categories: "intercom, ruby, mistakes"
---

Here's a quick way to move [Intercom][intercom] conversations from your test sandbox into production. Why would you ever want to do that? If you screw up your Rails deployment configurations and dev keys accidentally make it into production, you could end up with hundreds of customer conversations in your dev sandbox (ask me how I know!). The web ineterface doesn't have any quick way to fix this (why would they?), so I had to use their API to iterate through the open conversations and manually recreate them in production. There wasn't any information on how to do this to be found when Googling, and Intercom's API doesn't make this super easy, so I figured this post might someday help another unfortunate soul.

**Note:** There doesn't seem to be a way to recreate each message individually so I just joined them together into one string. It's not 100% seamless but it does in a pinch. Also, I don't think Intercom's API supports finding conversations that are unassigned so you may need to go through and assign them to an admin user before running this.

{% highlight ruby %}
require 'intercom' #You'll need the Intercom Ruby gem

INTERCOM_PRODUCTION = Intercom::Client.new(app_id: PRODUCTION_APP_ID, api_key: PRODUCTION_API_KEY)
INTERCOM_TEST = Intercom::Client.new(app_id: TEST_APP_ID, api_key: TEST_API_KEY)

#You'll need to assign all of the conversations to a single user before starting this.
open_convos = INTERCOM_TEST.conversations.find_all(type: 'admin', id: ADMIN_USER_ID, open: true, per_page: 25)
convo_ids = open_convos.map(&:id)

convo_ids.each do |id|
  convo = INTERCOM_TEST.conversations.find(id: id)
  message = convo.conversation_message.body
  if convo.conversation_parts.length > 0
  	convo.conversation_parts.each do |part|
  	  if part.author.id == convo.user.id
  	    part.body != nil ? message += part.body : message += " "
  	  end
  	end
  end
  INTERCOM_TEST.users.load(convo.user)
  INTERCOM_PRODUCTION.messages.create(from: { type: 'user', email: convo.user.email}, body: message)
  #This closes the original conversation to avoid any confusion.
  INTERCOM_TEST.conversations.reply(id: convo.id, message_type: 'close', type: 'admin', admin_id: ADMIN_USER_ID)
end

{% endhighlight %}

Hope this is useful for someone, someday!


[intercom]: https://www.intercom.io/