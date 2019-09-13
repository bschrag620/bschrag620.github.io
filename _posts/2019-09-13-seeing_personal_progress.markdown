---
layout: post
title:      "Seeing personal progress"
date:       2019-09-13 13:15:20 +0000
permalink:  seeing_personal_progress
---

Sometimes in the midst of learning new things, I can feel as though I'm not making any progress. While I might be implementing a feature that I've never done before, the constant of bug after bug, rereading of documentation, googling for examples can make it seem as though I still don't know what I'm doing. This week though, I was able to see it from a different angle.

## A past project
Before starting the curriculum with Flatiron, I had a personal project that I was working on in Python. It's a [model](https://github.com/bschrag620/PyRad) simulator that outputs radiative properties of gases, based on data downloaded from a repository called [HITRAN](https://hitran.org). If you aren't familiar with Python, here's an [article](https://www.coursereport.com/blog/ruby-vs-python-choosing-your-first-programming-language) that goes a bit more into detail about how it's different from Ruby. From my point of view, while there are structural differences to the two languages that are a bit beyond my useage level, the biggest difference is syntax.

This week, someone approached me about wanting a feature added to the app. I was ecsatic that someone might be getting some use from it other than myself, and their request didn't seem like much of an ask...

## Pandora's box
Well, at least it wouldn't have been much effort to add in, if I had built it with a proper framework. That's where it hit me to see the progress I have made. I had build methods, or functions, that were 10's-100's of lines long. Sometimes repeating these functions with minor changes to handle 1-off cases. Several functions need to make url requests to download a text file. Did I simply make 1 function to handle an abstract url request and error's if the request failed? Of course not!!! I hard coded each url request into it's respective function. When I have several functions that need to open a file and write out data to it, did I do this abstractly? Absolutely not! It's so easy to just copy and paste code from one function to the next so that much be the best route, right?!?!?

Don't let the bugs and trying to sort out new features and libraries make you think that you haven't made it anywhere yet. I don't think anyone gets to a point where coding comes without bugs. In the grander scheme of things, we all have learned how to better structure our code and projects, and that's really what it's all about. Better structure allows for future growth.
