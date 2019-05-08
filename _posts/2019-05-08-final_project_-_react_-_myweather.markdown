---
layout: post
title:      "Final Project - React - myWeather"
date:       2019-05-08 16:52:04 +0000
permalink:  final_project_-_react_-_myweather
---


This project has been a long and gradual process. For this project, I chose to revisit my original Ruby CLI project and make a weather app. The original Ruby app web scraped data from https://weather.com/ .  For this one, I wouldn't be web scraping but instead be utilizing existing API's for data.

# Selection of API
For starters, I needed to select an API to use for the project. There were several options out there. Accuweather, Wunderground, and NOAA all had existing API's in place. The first 2 both looked like excellent options but they each had a catch, money. I was hoping for a free option, which is exactly what NOAA offerred (https://www.weather.gov/documentation/services-web-api). 

The NOAA API has a two-step process for getting current weather:

1) GET coordinates based on latitude and longitude
2) use coordinates from step 1 to GET current conditions

This seemed easy enough but it requires having a latitude and longitude first. Obviously that isn't the most user friendly approach. I wanted something simple - user can enter a zip code, or perhaps a city/state combination and get their weather. It's beginning to seem like the free option might be a bit more work...

# More API's
To retrieve cooridinates, I utilized Google's geocoding API (https://developers.google.com/maps/documentation/geocoding/start). The API makes TONS of great info available at your fingertips. For the project, all I was going to need was some lat/long coordinates. That was easy enough. There was just one minor setback however. The location data was going to be getting stored in the server, which means there needs to be a unique identifier associated with each. What seemed to make the most sense was to use the zip code as the identifier. The gecoding API doesn't give a zip code when you feed it a city/state query. Time to test my google-fu and see what else was available.

I ended up finding another handy API (https://www.zipcodeapi.com/). Again, tons of great options are buried within this one, but I was just after a single thing - zip codes for a city/state. I got lucky as well that they have a developer kit that's free but comes with a limit of 10 requests/hr. Not a big deal for the sake of the project.

# Backend work
With the API's sourced, I built out a Rails API only backend. This was going to give a simple was to connect the React front end to these API's. I also utilized the backend to do the heavy lifting for gathering the weather and forecast data as well. This way, the React app stands as it's own individual app. If I wanted to switch weather API's down the road, I wouldn't need to touch anything in the React app. When users request data for a location, the process goes as follows:

1) GET request made to backend for existing location
2) Parse the string to determine if it's a city/state or zip code
3) If it's a zip code, proceed to 4. Else parse the city/state and retrieve a zip code from zipcodeapi.com
4) Return the locaiton data for the zip code. If not existing, return a zip code not found error 404
5) If 404 was received, then make a POST request to backend to create a new location based on the same entry

These 5 steps return an object that contains the necessary information and data to now make requests for weather. I could have simple fired the GET requests to NOAA from React but again, this is where I wanted the React app to not do any work. I also considered having the backend retrieve all the relevant weather and forecast data with the location object, but that seemed like to large of a step to take. Seperation of concerns was being violated, in my opinion. It also makes for a slower user experience. By breaking up the steps, I could keep the user better informated that the wheels are turning a progess is happening.

So once the location data is received back, a location card is updated with the relevant data. A message of 'Retrieving forecast data' is displayed on the card while the rest of the info is fetched. Three seperate ajax requests are fired to the backend:

1) Retrieve current conditions
2) Retrieve weekly forecast
3) Retrieve hourly forecast

The backend takes each of these requests are fires it's own GET request to the NOAA weather API. It handles parsing the data and returning only the needed information. As the front-end receives back the info, the display cards get updated, replacing the 'Retrieving data...' with the new content.

# Front end work
This is where I struggled, a lot! Going into creating the front-end, I really wasn't too sure where to start. My choices led me to having to nearly rebuild the enitre front end from scratch on a couple of occassions before I finally found the workflow that worked best for me. What ended up working best was:

1) Create the basic heirarchy of routes and components, leaving the components empty with the exception of maybe a bit of text telling you the component name
2) Utilize console.log to test buttons and GET requests that you are going to be making. This really helped to diagnose issues on the fly for me
3) With the layout and flow working properly from the user perspective, plug in the actions to the components to make it all come to life

# Final thoughts
The last several months I've spent going through the full stack web development curriculum has been time well spent. While I was coming into the program with some basic knowledge and understanding of coding there is a whole other world that I was introduced to. The biggest key that happened though was learning that I could to teach myself. 

The lessons and projects made a great framework to introduce me to new concepts, but more importantly in my opinion, they helped to teach the key basics that are necessary for students to continue their development and education. Being able to read through the documentation for a particular API, libracy, module, etc and understand it is what matters the most. 

As I look forward to a career in web development, it has become obvious through this course work that things simply aren't going to stay static in this industry. What I feel Flatiron has given me is the ability to learn and grow with that industry as it changes.

https://github.com/bschrag620/my_weather
