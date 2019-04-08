---
layout: post
title:      "Bike Warehouse - with JS"
date:       2019-04-08 18:03:48 +0000
permalink:  bike_warehouse_-_with_js
---


The Rails/JS project is a refactor of my previous project, Bike Warehouse. It wasn't a full refactor by any means, rather it was simply adding in some JS functionality to give a few pages a more dynamic feel. The refactor project itself had only a few requirements:
1. Make use of ES6 features (Arrow functions, const, let, etc...).
2. Render at least one index page dynamically
3. Render at least one show page dynamically
4. Render at least one has_many relationships to the page
5. Must use the Rails app to render a form for creating a resource that is dynamically submitted using JS
6. At least one of the JS models must have a method on prototype

# Using ES6 features

This was a rather simple request to meet. To accomplish this required properly declaring a few variables and utilizing arrow functions to build callbacks.

In JS, there are 3 ways to declare variables: *var*, *let*, and *const*. The latter two were added with ES6. Each of these are unique ways to define a variable with different uses.

*var* is a rather loose approach. I say this because the same variable name can be declared and later used anywhere within the code and it won't throw an error. This could lead to potential headaches since you might be referencing an already existing variable, for example within a function, that you declared globally using *var*. 

However, by using *let*, that conflict can be avoided since *let* will only let a variable be declared that isn't in use.
<a href="https://imgur.com/se9UH06"><img src="https://i.imgur.com/se9UH06.png" title="source: imgur.com" /></a>

The other key difference is that *let* only exists within the block that it is defined in. Once the end of the block is reached, the variable no longer exists.
<a href="https://imgur.com/9u1njSx"><img src="https://i.imgur.com/9u1njSx.png" title="source: imgur.com" /></a>
In this simple example, it can be seen that the *myName* variable is created within the block but no longer exists after the function ends. Taking the above example one step further by declaring *let* inside an *if* block, *myName* no longer exists once the *if* block has completed, even though we are still inside the function:
<a href="https://imgur.com/NDHGku1"><img src="https://i.imgur.com/NDHGku1.png" title="source: imgur.com" /></a>
One last edit, let's change the *let* variable declaration to *var*. Now we can see that the variable persisted within the function:
<a href="https://imgur.com/2fe1A7j"><img src="https://i.imgur.com/2fe1A7j.png" title="source: imgur.com" /></a>
It, however, doesn't exist once the function exits.

# Render dynamic index
To accomplish this, step one was to clear the existing .html.erb file to simply have a place for JS to hook into to display the information. What used to have an embedded function to iterate over the @bikes instance variable and display the relevant info, now simply contains:
```
<div id="table" class='bikes index'>
	<h1>Bikes</h1>
	<%= render 'table' %>
</div>
<script type='text/javascript'>
	init();
</script>

```

The partial view *table* that is rendered creates an empty table with the appropriate headers (depending on user permission level):
```
<table id='bikes-table'>
	<tr>
		<th>Manufacturer</th>
		<th>Frame</th>
		<% if !is_admin? %>
			<th>Size</th>
		<% end %>
		<th>Components</th>
		<th>Quantity</th>
		<th>Rating</th>
		<th>Price</th>
		<% if is_admin? %>
			<th>Part number</th>
			<th>Serial number</th>
			<th>Status</th>
			<th></th>
		<% end %>
		<th></th>
	</tr>
</table>
```
The final script tag is what initiates the JS to do it's thing. I had a bit of an issue with the table not always populating when I navigated to the page. I think the issue stemmed from the fact that if a page is cached, the *$(document).ready()* doesn't get triggered. Forcing a refresh would trigger the JS code to run and the table to be populated. However, what I found to be the simplest fix was to simply call an *init()* function from the bottom of the html. This insured that the page was loaded and the proper html tag's in place for the JS to hook into. The other approaches I attempted either left me with a page that wasn't consistently loading, or the JS functions being called before the elements were loaded, which still left me with an undesired result.
 
Within the *init()* function, a few things happen. Step 1, an ajax request is fired to retrieve all the bikes from the server. Once all bikes are received, they are added to the existing html table that was initially served to the DOM by Rails. The next thing that happens is some event listeners are added. For the index page, the event listeners are added to the headers of the index table. This allowed for some functionality to sort the table by the appropriate header if the user clicks on it.
Initial loading:
<a href="https://imgur.com/jwgMqHu"><img src="https://i.imgur.com/jwgMqHu.png" title="source: imgur.com" /></a>
After clicking on price:
<a href="https://imgur.com/qwYWx2s"><img src="https://i.imgur.com/qwYWx2s.png" title="source: imgur.com" /></a>
I mentioned earilier that the listeners were attached *if* it was the index page. If it happened to be the show page instead, different listeners were attached.
# Render dynamic show page
Similar to the index page, step 1 of this refactor was to remove almost all of the code that referenced the instance variable, @bike. I say almost all because I still needed a way to tell JS which bike it needed to gather information on. To do this, a *div* block was created for the page that had an *id* attribute of the bikes id in the database. This allowed JS to read the id on the appropriate *div* and fire the ajax request to get the info on the given bike.

Once the info was given, the rest of the process follows along somewhat similarly to the index page, simply populate the page, utilizing Jquery to find the appropriate element id's to fill in.
# Render has_many relationship
Part of the original Rails app was the ability for users to leave reviews on a bike. This meant that a bike has_many reviews. To pass the info from the server to the DOM, I utilized the ActiveModel::Serializer to convert the necessary info to JSON. To do so actually required a few serializers be created.

For starters, I needed a serializer on *reviews*:
<a href="https://imgur.com/a3OCUow"><img src="https://i.imgur.com/a3OCUow.png" title="source: imgur.com" /></a>
From this serializer, we can see that this JSON is going to belong_to a *bike*. The *bike* serializer then looked like:
<a href="https://imgur.com/JyARjb5"><img src="https://i.imgur.com/JyARjb5.png" title="source: imgur.com" /></a>
This takes care of passing on the reviews as an element within the bike JSON:
<a href="https://imgur.com/z0fgNgk"><img src="https://i.imgur.com/z0fgNgk.png" title="source: imgur.com" /></a>
From there, JS iterates over the reivews and fills out the **div#reviews** with individual review blocks with the relevant information.
# Dynamically submit a form
With the new JS version of bike_warehouse, the reviews form on a bike page can be submitted and the review will show up without a page refresh. Doing this required attaching an event listener to override the default functionalitiy of the form submit button:
```
function hijackFormSubmit() {
	$("#new_review").submit(function(e) {
		e.preventDefault();
		
		let data = $(this).serialize();

		$.post('/reviews', data).done(function(resp) {
			review = new Review(resp)
			prependReviewBlock(review)
		})
	})
}

```
Using preventDefault() keeps the page from firing the normal POST request to the server. Instead, the data that the user filled out becomes serialized and sent via an ajax post request to the server. Since this is handled via ajax, the user won't see a page refresh happen. Instead, once JS receives the response it creates a new JS model from the data and prepends it to the appropriate block. I chose to prepend the review so that it showed at the top of the reviews.
# At least one JS model
As I just mentioned, I implenmented a JS model and constructor for the reviews. This gave a clean way to handle the reviews as objects as well as provide them with their own individual methods. As it is, the database keeps track of when a review is created, but it isn't the most reader friendly way to present the information. So the reviews model has a prototype function to parse the updated_at string into a more user friendly, easy-to-read version.
<a href="https://imgur.com/6qVMBCz"><img src="https://i.imgur.com/6qVMBCz.png" title="source: imgur.com" /></a>
While there is probably a better way to parse the data, this served the purpose of what was needed. Now, when the review blocks are added to the page, the users are left with a bit easier to read date than the original updated_at string that is stored:
<a href="https://imgur.com/3Ff21Na"><img src="https://i.imgur.com/3Ff21Na.png" title="source: imgur.com" /></a>

