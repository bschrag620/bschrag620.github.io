---
layout: post
title:      "Creading PDF's with Rails"
date:       2019-09-07 13:05:17 +0000
permalink:  creading_pdfs_with_rails
---


[Last](http://bradschraglearns.com/never_stop_learning) week I discussed how to setup emails. Part of what I wanted to accomplish in that project was to attach a pdf to the email. There are several gems that can assist with this. I chose to go with [wicked-pdf](https://github.com/mileszs/wicked_pdf). 

## Basic setup
`wicked_pdf` acts as a wrapper[ wkhtmlbinarytopdf-library](https://github.com/zakird/wkhtmltopdf_binary_gem), so to get all this started, make sure you have both those in your `Gemfile` and then run `bundle install`:

```
gem 'wicked_pdf'
gem 'wkhtmlbinarytopdf-library'
```

After install, be sure to generate the initializer for `wicked_pdf`
```
rails g wicked_pdf
```
And that's it, time to make some PDF's!

## Creating a layout
In a similar fashion to how you setup an `application.html.erb` layout to handle loading the head, body, and footers, etc. for your views, you'll need to setup at least one `pdf.html.erb` layout. Part of what needs to be specified in it is the stylesheet. For my needs, I didn't need to setup different styling for the pdf, so it was very straight forward:

```
<!DOCTYPE html>
<html>
<head>
	<title>PDFs - Ruby on Rails</title>
	<%= wicked_pdf_stylesheet_link_tag "application" %>
</head>
<body>
	<%= yield %>
</body>
</html>
```

Notice the use of `wicked_pdf_stylesheet_link_tag` rather than `stylesheet_link_tag` used for loading CSS in the `application.html.erb` layout.

## Generating a PDF
One aspect I really like about `wicked_pdf` was that you could easily use existing views as the layouts for a pdf. We just need to turn the view into a string using `render_to_string` and pass it into the pdf generator. It might look something like this:

```
pdf = WickedPdf.new.pdf_from_string(
			render_to_string(
				template: 'entries/show.html.erb',
				layout: 'pdf.html'),
			page_size: 'A4',
			orientation: 'Portrait',
			lowquality: true,
			zoom: 1,
			dpi: 75)
```

By simply specifying the template, wicked will use that view. We also need to be sure we tell it *which* layout to use. 

## Now what?
So what this block of code above does is returns a string that represents a pdf file, but it isn't a file just yet. Time to create a file and push the string into it. I used a temporary directory as I wasn't wanting to persist the pdf's on the server or have them linked to any of my models.

```
path_to_file = Rails.root.join('app', 'tmp', "#{id}.pdf")
		File.open(save_path, 'wb') do |file|
			file << pdf
		end
```

My method that generated the pdf had a return value (if successfully created and saved) of the file path. From there, I could make the pdf available for download or to attach. Going back to last week's talk about generating emails, here's how to attach that file to an email. In the `mailer` that we want to use (remember, it acts very similar to how a controller handles views), my method, or route if you will, took in the file path as an argument:

```
def new_wiring_sheet(attachment_path)
		attachments['data_sheet.pdf'] = File.read(attachment_path)
		mail(to: 'brad.schrag@gmail.com, subject: 'new data sheet')
end
```

## Don't stop learning
Finishing up the coursework at Flatiron is really just the beginning of the next step. The next step is going to rely on you to continue your education. Look for any opportunities you can, they are all around you.






