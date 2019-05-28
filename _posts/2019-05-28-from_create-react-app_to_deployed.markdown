---
layout: post
title:      "From create-react-app to deployed"
date:       2019-05-28 14:09:46 +0000
permalink:  from_create-react-app_to_deployed
---


With my final project getting wrapped up, I wanted my weather app to be deployed out on the web for access to everyone. Production and deployment is one aspect that the curriculum didn't cover. However, how to learn is what was covered, so off to find the answers. Of course others have done this, and as such have documented their efforts. I ran across this great tutorial which covered about 80% of what I needed to get my app deployed:

https://medium.com/jeremy-gottfrieds-tech-blog/tutorial-how-to-deploy-a-production-react-app-to-heroku-c4831dfcfa08

Most of the beginning part of this guide is the initial setup of the app, the info I needed starts on step 11.

# Setup for Heroku
Since the main app is a node.js application, we need to have a `package.json` file in the root folder for our application. Through the initial setup of the app, there is a `package.json` file that was created, however, it is in the `/client` folder. While this file is still needed to build our app, it isn't helpful to our deployment.

As Bruno mentions in his post, this file can be easily made by simply using `npm init` in the root directory which is the route I opted for. You'll be asked a series of questions pertaining to the app after which point the file will be created. 

After the file is created, open it up and add in the appropriate `scripts` values:
```

"scripts": {
    "build": "cd client && npm install && npm run build && cd ..",
    "deploy": "cp -a client/build/. public/",
    "postinstall": "npm run build && npm run deploy && echo 'Client built!'"
  }
```

While this may seem confusing, it's really fairly straight forward once you've walked through it a couple times. As Bruno mentions, `postinstall` magic is what we are going to take advantage of. `postinstall` is executed immediately after `npm install`, so let's look at each of these individually:

**postinstall**: This is the script that gets run. It's going to do 3 things.
First `npm run build` - run the build script
Second `npm run deploy` - run the deploy script
Third `echo 'Client built!` - let us know it was all completed
Seems simple enough, let's look at the individual scripts it is going to execute now.

**build**: When `postinstall` runs `npm run build`, it's going to change to the `client` directory (`cd client &&...` ) , then it's going to install the `npm` package in that directory (`npm install && ...`). This is the file you are probably most familiar with as it's the one that lists your app's dependencies. After the install is finished, it will run `npm run build &&...`. This is building a production version of the app and writing it to `/client/build`. Final step that the build script runs is to move back to the original home directory `cd ..`

**Deploy**: Upon deploying, heroku is going to copy the built app to the `/public` folder, hence `cp -a client/build/. public/`

Creating this `package.json` in the root folder is the first step in getting your app deployed.

# Creating a heroku app
To make all of this work, you'll need to create a heroku app. To install their CLI, check here:
https://devcenter.heroku.com/articles/heroku-cli
You'll need to create a user account on their website. You can also initialize your app their if you like. But if you want a wacky heroku name for your app, simple run `heroku apps:create` in the root folder of your application (such as https://arcane-river-19081.herokuapp.com/). This is somewhat similar to initiliazing a folder as a git repository. Next, we need to setup the buildpacks for the app.

# Buildpacks
Buildpacks tell heroku how exactly to build the app so it can be properly run on the dynos. Bruno, again, does a great job of touching on the details of this. To setup the buildpacks, simply run these two commands:

```
heroku buildpacks:add heroku/nodejs --index 1
heroku buildpacks:add heroku/ruby --index 2
```

This basically says: 
**Step 1** - build as a nodejs app
**Step 2** - build as a ruby app
Heroku will expect to find a `package.json` in the root folder for step 1 and a `Gemfile` for step 2.  The final new file we need is a `Procfile` in the root of our appication folder. In that `Procfile` we need 1 line:
```
web: bundle exec rails s
```
This will get our rails server running. As a test, try running `rake start:production`. This should execute all the scripts we've made up to this point. Assuming it all worked, you now have a `/public` folder that was created, so let's add a line to `.gitignore` so we don't push it to our repo.

```
/public
```
# Postgresql
Heroku won't use the `sqlite3` gem, in fact sqlite3 isn't really meant for production at all. It's great for easy and quick setup of a local environment. You'll need to transiition your app to use the `pg` gem. Thankfully, heroku has a great writeup on how to do that. 
https://devcenter.heroku.com/articles/sqlite3
I did hit a couple hiccups along the way, so I'll try to touch on those.

First off, to use `postgres`, you need it installed on your machine. A quick google will reveal:
http://postgresguide.com/setup/install.html

Once installed, you'll need to start the `postgres` service. On Linux, that looks something like `sudo service postgresql start`

I ran into a bit of a snag when trying to execute `rake db:create && rake db:migrate` due to some permission errors. Postgres has a console that allows you to add and delete users as well as their permissions. To access it (again for Linux users), run `sudo -i -u postgres psql`. The `-i` and `-u` flags will run the command as if we are the `postgres` user. `psql` is actually the command that runs the postgres console.  If all has gone well, you should be seeing a console that looks like:
```
postgres=# 
```
From here, `\du` will list out the users and their permissions. I needed to add login as a user and update it's roles to include `CREATE_DB`. Here's some links to the postgres documentation for adding and editing users:

https://www.postgresql.org/docs/8.0/sql-createuser.html
https://www.postgresql.org/docs/8.0/sql-alteruser.html

Hopefully, with that all said and done, you can successfully run `rake db:drop && rake db:create && rake db:migrate`. If so, you are 1 step closer to deployment.

# Pushing all the changes
After creating some new files, and migrating over to postgres, you should be ready to get this app launched. Remember how I said running `heroku apps:create` was kind of like initializing your folder as a git repository? Well that's because you really kind of did just that. Now instead of pushing to `origin` we are going to push these updates to `heroku` by running:
```
git add .
git commit -m 'deploy'
git push heroku master
```

After that, you should see in your console the files getting pushed and the app getting built. Assuming no errors, run:
`heroku run rake db:migrate` and if you had a seed file `heroku run rake db:seed`. This will get your db setup on heroku's end.

# One minor snag
So about here I ran into an issue with the requests to external api's not working. Finally after a bit of debugging it became evident that my api keys weren't being used. Why was that? I had them setup in my `.env` file, which I made sure to include in my `.gitignore`.... which of course means they aren't getting pushed to external repos! So my keys aren't being pushed to heroku. To add your keys and environment variables to the heroku app, you'll need to use the heroku CLI (https://devcenter.heroku.com/articles/config-vars). Simply run the command:

`heroku config:set SOME_API_KEY_FROM_DOTENV_FILE=the_api_key_value_here`

Run the above command for each of the enviornment variables in your `.env` file. This keeps your keys protected from the public and scary world of the interwebs.

# Routing
The last snag that you might run into deals with routing. Bruno covers this as a bit of an afterthought that came up from a question on his blog. Basically, the rails router is going to be receiving all the incoming hits. So we need to route anything that isn't associated with our rails api to the react app.

First, in `application_controller.rb`, we need to create a fallback method:
```
class ApplicationController < ActionController::Base
  def fallback_index_html
    render :file => 'public/index.html'
  end
end
```

**Also right here**, make sure your `ApplicationController` is inheriting from `ActionController::Base`, not `ActionController::Api`. I had a bug with some of the custom routes that ended up being due to it pulling from api instead of base. If you build your app using `rails new my_app --api`, this is probably still setup to pull from `ActionController::Api`.

Second, in `routes.rb`, make sure we send any unrecognized path to the newly created fallback method in our application_controller:

```
get '*path', to: "application#fallback_index_html", constraints: ->(request) do
    !request.xhr? && request.format.html?
end
```
The `constraints` are simply making sure that if the path is not an xhr request and is asking for html, then send it to the fallback method.

# Conclusion
At first glance, it seemed like a lot of work to get an app deployed. In hindsight, while I did hit a couple snags, it was rather simple. The snags that I did run into provided great opportunities to further my understanding of these processes so it was well worth it. My app can currently be found here:

https://arcane-river-19081.herokuapp.com/
