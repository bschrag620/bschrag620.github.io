---
layout: post
title:      "Downloadable content via Rails"
date:       2019-09-20 18:49:00 +0000
permalink:  downloadable_content_via_rails
---


Today I'm going to rehash my theme from a couple weeks ago, don't stop learning, and cover the topic of making content downloadable. Through the Flatiron coursework, this was one of those things not touched on that I quickly found a need for. Making a link stream data in Rails is super easy!

## send_data
For what I was doing, I wanted users to be able to click a link and download a pdf. I covered previoiusly how to generate the pdf, as well as attach the pdf to an email. Now let's look at the code to allow the user to download it. For starters, I setup a route:
```
./config/routes.rb

...

get '/forms/:id/download', to: 'forms/download', as: 'form_download'

...

```


Simple enough start. Next, add the method into the controller:

```
class FormsController << ApplicationController

...


def download
		pdf_path = generate_pdf(params[:id])

		file = File.open(Rails.root.join('public', 'uploads', pdf_path), "rb")
		contents = file.read
		file.close

		File.delete(pdf_path) if File.exist?(pdf_path)

		send_data(contents, :filename => "Form_data_#{params[:id]}.pdf")
	end
	
	...
```

Let's breakdown what happens here.
1) Create a pdf using the generate_pdf method. The return value of generate_pdf (assuming it was successful), is the path to the new file.
2) Open the file in read-byte mode ('rb') and store it's value into a variable, `contents`
3) Close the file and delete it. Adding the conditional `if` on the `File.delete` insures that nothing will crash if the file happens to not be there.
4) Send the data, using `#send_data` with the filename "Form_data_(form_id).pdf"

## Seems too easy
And really, it is. Let's take a closer look though at what Rails is doing in the background to make this happen. First off, [`#send_data`](https://apidock.com/rails/ActionController/Streaming/send_data):

```
# File actionpack/lib/action_controller/metal/streaming.rb, line 112
      def send_data(data, options = {}) #:doc:
        send_file_headers! options.dup
        render options.slice(:status, :content_type).merge(:text => data)
      end
```

Breaking this down, we can see first line duplicates the options, and passes it to a method called `#send_file_headers!`. One point of interest here is that it is using `.dup` rather than just sending it options. Reason for that is `#send_file_headers!` makes changes to the `options` object passed to it, that will change the original `options`. In other words, it might be a bit destructive. Let's see what that looks like, firing up `irb`, give this a go:

```
2.6.3 :001 > my_props = {:name => 'brad', :age => 39}
 => {:name=>"brad", :age=>39} 
2.6.3 :002 > def update_with_height(prop)
2.6.3 :003?>   prop.update(:height => 72)
2.6.3 :004?>   end
 => :update_with_height 
2.6.3 :005 > my_props_with_height = update_with_height(my_props)
 => {:name=>"brad", :age=>39, :height=>72} 
2.6.3 :006 > my_props
 => {:name=>"brad", :age=>39, :height=>72}
```
Because I didn't send a duplicate of `my_props` it was updated destructively by the method. Had I sent `my_props.dup` instead, my return value would have been the same, but `my_props` would have remained unchanged.

#### Functional programming... a quick side not
This is one important aspect of functional programming, and that's that functions or methods that are called are not destructive to the original values that were passed to them. Whether that means passing a duplicate object value in Ruby, or utilizing `.slice` in JS rather than `.shift/.unshift/.pop/.push`, the main point is that the functions don't have any side effects. They always return the same value when passed the same initial parameters and they are not destructive. No weird side effects!

## Back to our originally scheduled program
So the following line is a call to `render`. Again, being non-destructive but only passing the necessary info, `.slice` gets called on our options to only send the relevant properties. This leaves `options` untouched. Going back to my previous example:

```
2.6.3 :011 > my_props_with_height.slice(:age)
 => {:age=>39} 
2.6.3 :012 > my_props_with_height
 => {:name=>"brad", :age=>39, :height=>72}
```

`my_props_with_height` doesn't get changed when `.slice` is used. The data that is sent (`contents` variable in the case of my code), is sent as `:text`.

## So how did the browser know to download rather than render?
Ahh, so that actually happened because of the first method that I glossed over, `#send_file_headers!`. Turns out, if you crack open the code for it, one of the things you'll see is a global variable:

`DEFAULT_SEND_FILE_OPTIONS = {
      :type         => 'application/octet-stream'.freeze,
      :disposition  => 'attachment'.freeze,
    }.freeze`
		
		The user passed `options` get merged with `DEFAULT_SEND_FILE_OPTIONS` via `.merge` (remember, don't have any weird side effects). That `:dispoisition` key with a value of attachment, that's what tells the browser specifically to download the content as a file, rather than display it. The `:type` tells the browser, I don't really know what kind of file this is, but it will be binary data. It's best to specify in your options what kind of data you are sending (ie `application/pdf` would have been good for my example).
		
		RoR send_file_headers: https://apidock.com/rails/ActionController/Streaming/send_file_headers%21
		RoR send_data: https://apidock.com/rails/ActionController/Streaming/send_data
