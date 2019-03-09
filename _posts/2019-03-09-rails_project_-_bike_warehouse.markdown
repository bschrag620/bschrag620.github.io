---
layout: post
title:      "Rails Project - Bike Warehouse"
date:       2019-03-09 15:46:56 +0000
permalink:  rails_project_-_bike_warehouse
---

For this project, I chose to tackle sort of Amazon style website where users could login , purchase, and review bikes (after all, I'm a bit of a cyclist).  For me, what seems to be a rather simple project escalates rather quickly. The project ended up with 11 controllers and a namespaced admin controller that contained another 6 controllers of it's own.

The project has been very educational. One key aspect I picked up on was in the namespacing of routes. As I mentioned, the admin controller became a namespaced route and controller. There are several advantages to namespacing a route:

* Easy control over access to the controller - In my users and pruchases controller, it became necessary to validate that the current_user matched up with the user info that was being requested through params. This was easy enough to accomplish by construcing a method in the ApplicationController then simply calling that method each time it was needed. Typing out a single line of code isn't terrible of course, but when done on every controller action, in a handful of controllers it can become a bit tedious. My bigger concern is what if - What if I forget to implement the check on one of the actions? Now there is a vulnerability in the code. By namespacing the controller, I was able to create a BaseController that the admin controllers inherited from. In the BaseController, I simply put in a callback for before_filter :is_admin? where is_admin? was a simple private method to validate the current_user as an admin. All the controller actions that inherited from BaseController will now validate the current_user is an admin.

* Easy control over layouts - This is one aspect that was covered in the lessons leading up to this project that didn't sink in until I was doing this project on my own. It really follows the same idea as easily providing access. It's not difficult to specifiy in a controller which layout to use. For the one-off cases, it makes sense to override the default layout. However, one the number of controllers starts to grow and the case become 2-off or 3-off, it's time to rethink what you are doing. Namespacing the routes and controllers MIGHT be a better approach. 

* Downsides to namespacing - More folders. This initially seemed like a real pain. When I was building views I was needing to setup a whole list of folders and it felt repeatative. In hindsight, this really isn't bad. It really helped to clean up the code.

If I were going to do this project over again from the ground up, I would:

1. Have put a bit more planning into the general layout and structure of my app
2. Would have namespaced the user routes and controller
3. Potentially namespaced a controller to be used when the user isn't logged in

https://github.com/bschrag620/bike_warehouse


