---
layout: post
title:      "Never stop learning"
date:       2019-08-31 03:49:08 +0000
permalink:  never_stop_learning
---


Simple words of advice. While I’m still very early into a career search, I’ve reached out to a couple individuals who are former students at Flatiron for their personal advice. The most recent tip I received was “Never stop learning”. Those few words gave me a bit of a confidence boost. 

I’ve always considered myself a curious individual. I enjoy learning new things and am always looking for ways to extend my abilities. While learning new things, be it coding or some other useful (ok, often what I learn isn’t really useful) skills, the best thing I feel I can do with that is to pass that info on so others can prosper from it as well. With that in mind, my next couple blog entries are going to focus on life since Flatiron and a couple of the new things I’ve learned that weren’t directly part of the curriculum.

### Generating emails with Rails

Shortly after wrapping up the coursework at Flatiron, my supervisor at work asked if I could help sort out some details of an online form builder they use for storing details of work that was completed at a job site. The presentation of the info that was captured wasn’t very ideal for how it was used later on. So my task was to see if we could somehow alter that presentation.

Unfortunately, the form builder didn’t have a good way to change the output of the data. However, it did have a POST option. When forms are submitted, along with saving in the database on their end (the form builder website), the data can also be sent to some other website. Now if only I had the ability to create a server capable of receiving information….

I filled in my supervisor that it would probably take the better part of a day or two to finish, but I had a good solution in mind that was going to give a lot of flexibility to future changes. First steps were to create a new Rails app, setup a couple routes, update the form bulider with the new apps POST route, and voila, the form was submitting it’s data to my new app. That was the easy part, the next part took a bit of research – how to generate and send emails via Rails.

### Setting up ActionMailer

First things first, you need to setup a couple environment variables. I took the route of using a gmail account, but whatever type of mail service you will be using, you need to store that data. Of course you don’t want that data leaked, so be mindful of not pushing the file to github. I used the dotenv gem to manage the environment variables. So inside my .env file I setup:

```
GMAIL_USERNAME=brad.schrag@gmail.com
GMAIL_PASSWORD=my_super Secr3t P@ssw0rD!!! 
```
(hopefully it’s obvious this isn’t my password, but knock yourself out if you want to give it a go)

Next, setup your production environment to use the gmail info, in ```/config/environments/production.rb```

```
config.action_mailer.smtp_settings = {
    address: "smtp.gmail.com",
    port: 587,
    domain: “THE_HOME_ADDRESS_OF_YOUR_SERVER”,
    authentication: "plain",
    enable_starttls_auto: true,
    user_name: ENV["GMAIL_USERNAME"],
    password: ENV["GMAIL_PASSWORD"]
  }
	```

After setting up the config, I generated a new Rails mailer:

```rails g mailer custom_mailer```

In similar fashion to how a controller will have it’s own folder in /views, your new mailer will have a forlder in views as well. In the same way a controller has different methods (show, edit, new, etc) that route to their respective views, the mailer will have methods that route to their respective views as well:

```class CustomMailer < ApplicationMailer

	def new_data
		mail(to: 'brad.schrag@gmail.com, subject: 'new data')
	end

	def update_data
		mail(to: 'brad.schrag@gmail.com', subject: 'updated data')
	end
end
```

So in this example above, Rails will expect to find a ```views/custom_mailer/new_data.html.erb``` and ```views/custom_mailer/update_data.html.erb```. Of course if you need to, you can create some environment variable, and pass those on to the mailer views, just as you would do in a normal controller method.

Last, but not least of course, is making Rails actually send an email. To do that, it’s as simple as:

```CustomMailer.new_data.deliver```

Next week, I’ll dive into creating a pdf and attaching it to these emails. Until then, don’t stop learning.nt of your blog post goes here.
