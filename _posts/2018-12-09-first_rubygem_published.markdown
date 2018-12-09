---
layout: post
title:      "First RubyGem published"
date:       2018-12-09 07:27:20 +0000
permalink:  first_rubygem_published
---


So I've officially made my first RubyGem. It's based on a menu system that I've made in Python. Despite the similarities there were still a few hurdles to overcome.

In Python let's say we have a function, or method, foo
```
def foo
    prints('bar')

```

To call functions in Python, you then
```
foo()

> "bar"
```

We could also assign this function to a variable by simply assigning it as you would any other value:
```
myBar = foo
```

Since there aren't () after foo, it doesn't call the function, only assigns it. Ruby, however, doesn't require paren to launch the method, so assigning to variables isn't done the same way. Instead, if I had the same #foo method in Ruby, to assign it :
```
my_bar = method(:foo)
```

This will actually setup my_bar to be a pointer to the method. Then whenever you actually want to call the method:

```
my_bar.call
```

The .call will explicitly execute the applicable method. 

The RubyGem I made relies heavily on this concept of assigning functions or methods to an instance variable. When a user selects an Entry from a Menu, it executes .call on the Entry objects next_function variable. This could be another Menu object or maybe just another #method to retrieve or manipulate some data. 

So if you are in need of a simple, text-based, nested menu system in Ruby, definitely try this one out!

https://github.com/bschrag620/gem_menu
