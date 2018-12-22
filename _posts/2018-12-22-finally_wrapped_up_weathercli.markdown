---
layout: post
title:      "Finally wrapped up weatherCLI"
date:       2018-12-22 21:44:21 +0000
permalink:  finally_wrapped_up_weathercli
---


So got my weatherCLI wrapped up.  As usual, took me a bit longer than I expected. Didn't help that life outside of coding happens. Jumping back into the project midway required a couple days to readjust.

The goal of the project was to pull weather data, in varying formats, from weather.com for a given zip code. To accomplish this there are several classes and modules that I created. For starters, #GatherData module contains classes for each type of forecast that can be pulled from weather.com (5 day, single detailed, hourly). Each of these classes has a unique URL that gets passed to the #return_html method. The #return_html method uses the default URL and zipcode to gather the corrrect data. It also handles the 404 case if information isn't found.

The returned html document is passed the appropriate class within the ParseData module. These classes utilize Nokogiri to parse the html and return the appropriate values for each type of forecast the user is requesting to view. It returns the values in a hash. Depending on the type of forecast requested, it might be an array of hashes, as a couple display options display multiple forecasts at once.

These hashes are used to initialize Forecast::Day objects. The Day class acts as a storage container for all the values to be tracked (max_temp, min_temp, current_temp, wind, etc). It also has a #display method which generates WeatherCards for it's representative type. WeatherCards vary in size and information displayed, so each is a bit unique.

Finally, to tie all this together, I created a Ruby Gem ([gem_menu](https://rubygems.org/gems/gem_menu)) to that allows a user menu interface to be easily constructed. gem_menu has Entry classes, which are made up of a description, next method, and parameters to be passed on. It also has a Menu class which is made up of a title, array of entries to be displayed, as well as an optional previous menu. Here's an example of using gem_menu to create the "Choose forecast" menu in weatherCLI:

```
def choose_forecast(zip)
        entries = []
        entries << GemMenu::Entry.new("5 day forecast", method(:display_5_day), {:parameters => zip})
        entries << GemMenu::Entry.new("Single day detail", method(:display_single_detail), {:parameters => zip})
        entries << GemMenu::Entry.new("Hourly", method(:display_hourly), {:parameters => zip})
        optional = {:previous_menu => method(:main_menu)}
        GemMenu::Menu.new("Choose forecast to view", entries, optional_args=optional)
    end

```

Each entry is passing on the zip code that was selected in #main_menu by the user to a method that calls the correct display type. Example of #display_5_day:

```
def display_5_day(zip)
        WeatherCLI::FiveDay.display(zip)
        choose_forecast(zip)
    end

```

One last bit that was created for this was a way to track user entered zip codes, and store them for next time. This way, users don't have to continuously type in a zip code to see information. To do this, a Settings singleton was created, which reads and writes weatherCLI parameters to a settings file in a JSON format. When a user adds a zip code, it gets written to the ./config/settings file. Next time they launch weatherCLI, all the previously saved zip codes are loaded.
