---
layout: post
title:      "Sinatra web app"
date:       2019-02-03 04:10:19 +0000
permalink:  sinatra_web_app
---


So for this project I chose to right an app to track users for a non-profit organization I've been a part of. The goal was to not only give a framwork for a website, but more importantly give a way to check to see if users are authorized to use equipment. 

The NWA Creative Hub is a makers space that has a few different pieces of equipment that is open to the public for use. However, some of the equipment can be dangerous, some of it fragile if used imporperly. So to make sure only people that know how to use it are accessing it, we had the idea a while ago to integrate some sort of certificates into the user's account. Using a raspi or an arduino as a control board, users could scan a badge, the board could make a GET request, and if the user is certified, power up the equipment for use. There are tons of other handy metrics here too which could be built in:

* which equipment is used most
* maintenance interval for equipment
* who uses what the most often, which can help if there is another individual needing guidance
* etc

For the app, there are several tables and relationships that were implemented. The trickiest for me was actually the Test model and it's tables and relationships. 

Tests are made up of many Questions. Questions have many answers. Tests belong to a Certificate. To manage all these required implementing a couple extra models to handle the join tables, but once sorted out it works quite well. 

For me during this project, I think one of the key takeaways was the importance of utilizing `rake db:seed` to handle setting up the database with some basic info. Early on, as I was making mistakes creating users, tests, etc. I spent quite a bit of time simple deleting the database, `rake db:migrate` to set it up again, and then repopulating it through the site's forms I had created. Much time would have been saved had I just utilized a seed file from the beginning.  But then again, that's the point of this type of exercise, learn from the mistakes. 

https://github.com/bschrag620/creative_hub
