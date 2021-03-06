# Isotope - Ruby Hybrid Template Engine for Client Side and Server Side

## The problem:

In Ajax-based sites, there's a constant dilemma: How to get objects rendered in templates? In server side (and output full HTML)? Client side (and mess with JSON objects)?
What should be the role division between server and client?

## Common Approaches, Pros & Cons:

A few approaches to output a rendered template evaluated with an object are:

### Approach #1: Regular ERB Partial

Evaluate a simple ERB partial with a local object, and server it as a **string** to the client, simply by

	<%= render :partial => "article", :object => @article %>

in a view, or from a controller and request it by Ajax.

This partial can look like:
	
	<h2><%=article.title%></h2>
	<div class="content">
	<%=article.content%>
	</div>
	<ul class="tags">
		<%article.tags.each { |tag|%>
		<li><%=tag.name%></li>
		<%}%>
	</ul>
		
#### Pros

* Simple, readable and well known ERB for Rails/Sinatra
* SEO and accessibility - HTML code is downloaded into the source

#### Cons

* Server side only
* Requires download of the whole HTML code, can cause performance issues
* Cannot bind easily to a different object on client side. Must re-rendered in the server-side and be downloaded


### Approach #2: Client Side EJS Template with JSON Objects

Having an EJS template in the HTML code, with techniques such as John Resig's [JavaScript Micro-Templating](http://ejohn.org/blog/javascript-micro-templating/)

	<script type="text/html" id="article-template">
	<h2><%=item.title%></h2>

	<div class="content">
	<%=item.content%>
	</div>

	<ul class="tags">
		<%item.tags.forEach(function (tag) {%>
		<li><%=tag.name%></li>
		<%});%>
	</ul>
	</script>

Ask the server for a JSON article and evaluate the template with this object into a string, and place it inside a container, using a techniques as mentioned:

	var results = document.getElementById("results"); // some container on the page
	results.innerHTML = tmpl("article-template", article); // article is an object, probably a result of an AJAX JSON request

#### Pros

* Fast - requires the server to send only the JSON object and the HTML is downloaded only once as a template

#### Cons

* SEO and accessibility - HTML code isn't in the source of the page but being rendered after load


### Approach #3: Regular ERB Partial With Marked Spots for Data Place Holders

This approach tries to combine server side and client side but requires a lot of work. It contains a regular ERB template and place holder markers like class names on elements.
The template can be first evaluated on the server with a Ruby object and on the client side it can be evaluated with a different JS object (probably from a JSON request).

Template file should look like:

	<h2 class="data-title"><%=article.title%></h2>
	<div class="content data-content">
	<%=article.content%>
	</div>
	<ul class="tags data-tags">
		<%article.tags.each { |tag|%>
		<li><%=tag.name%></li>
		<%}%>
	</ul>

And then, from a JS function, doing something like:

	// article is an object probably from a JSON request
	container.querySelector('.data-title').textContent=article.title;
	container.querySelector('.data-content').textContent=article.content;
	var tags=container.querySelector('.data-tags');
	tags.innerHTML="";
	article.tags.forEach(function (tag) {
		tags.appendChild(document.createElement("li")).textContent=tag.name;
	});

#### Pros

* One template for both client and server (not really, see Cons)
* SEO and accessibility - HTML code is downloaded into the source

#### Cons

* On the client side - Must mimic the Ruby functionality with JS when it comes to loops, conditions etc. However, text values are pretty easy to embed. This code should probably written manually for everything that is not a simple textual content.
* Must maintain data place holder markers


### So...

In these three approaches, the developer needs to choose according to the task and the project requirements, or worse, maintaining two templates, ERB and EJS.

Each approach is written in a totally different way, and switching between the approaches means a lot of work.


## Introducing: Isotope - Ruby Hybrid Template Engine for Client Side and Server Side

So why not combining all the **pros** of the approaches together?

The biggest constraints to be considered are:

* Client side doesn't understand Ruby
* Ruby can't be translated fully into JavaScript
* And the most important one: **Template should be maintained in one single file for both client and server uses**

Isotope is from greek - "Equal Place". An equal place of editing a template for both client and server (Thanks @yuvalraz for the name!).

Using [**jbarnette**](http://github.com/jbarnette/)'s AWESOME [**Johnson**](http://github.com/jbarnette/johnson/) gem, Ruby and JavaScript can interact together!

That means, that ruby code can handle EJS templates and JSON objects. A **great and very inspiring article** is [Write your Rails view in… JavaScript?](http://tenderlovemaking.com/2008/05/06/write-your-rails-view-in-javascript/) by [Aaron Patterson](http://tenderlovemaking.com/).

In this approach, only **one template is written and maintained in an EJS format**, for both client side and server side.

## Usage

Isotope gives the ability to have a single template file, and easily switch between the approaches:

	# article.ejs
	
	<h2><%=item.title%></h2>

	<div class="content">
	<%=item.content%>
	</div>

	<ul class="tags">
		<%item.tags.forEach(function (tag) {%>
		<li><%=tag.name%></li>
		<%});%>
	</ul>

### On the Client Side

Outputting from the server side (controller or view)
	
	<%= Isotope.render_template("full/path/to/article.ejs", :id => "article-template") %>
	
*Notice: a full path should be sent as the first variable, so either use `File.join(File.dirname(__FILE__), '../relative/path/to/article.ejs')` or with `Rails.root.join('app/views/articles/article.ejs')`*

The above code will output:

	<script type="text/x-isotope" id="article-template">
	<h2><%=item.title%></h2>

	<div class="content">
	<%=item.content%>
	</div>

	<ul class="tags">
		<%item.tags.forEach(function (tag) {%>
		<li><%=tag.name%></li>
		<%});%>
	</ul>
	</script>

which is easy to evaluate with any JS object using the [mentioned technique](http://ejohn.org/blog/javascript-micro-templating/).

### On the Server Side (The Holy Grail)

Using [Johnson](http://github.com/jbarnette/johnson/), the famous [micro-templating technique](http://ejohn.org/blog/javascript-micro-templating/) and JSONed Ruby objects, this library provides the following functionality:

	<%= Isotope.render_partial("full/path/to/article.js", :locals => { :item => @article }) %>

*Notice: a full path should be sent as the first variable, so either use `File.join(File.dirname(__FILE__), '../relative/path/to/article.ejs')` or with `Rails.root.join('app/views/articles/article.ejs')`*

This code reads the source of the EJS file, uses Johnson and John Resig's technique and serves a **string** as an output.


### Installation:

	gem install isotope

Or
	# Rails 3.x
	ruby script/rails plugin install git@github.com:elado/isotope.git
	
	# Rails 2.3.x
	ruby script/plugin install git@github.com:elado/isotope.git


#### Rails

##### Rails 3.x

Add to your Gemfile

	gem 'json'
	gem 'johnson'
	gem 'isotope'

and run `bundle install`

##### Rails 2.3.x

Add to config/environment.rb

	config.gem 'json'
	config.gem 'johnson'
	config.gem 'isotope'

and run `rake gems:install`

##### Server Side Example:

	# ArticlesController
	
	def show
		@article = {
			:title => "Hello!",
			:content => "World!",
			:tags => [
				{:name => "tag 1"},
				{:name => "tag 2"},
				{:name => "tag 3"},
				{:name => "tag 4"}
			]
		} # Or an ActiveRecord fetch
		
		render :text => Isotope.render_partial(Rails.root.join('app/views/articles/article.ejs'), :locals => { :item => @article })
	end

Or, with a view:

	# ArticlesController

	def show
		@article = {
			:title => "Hello!",
			:content => "World!",
			:tags => [
				{:name => "tag 1"},
				{:name => "tag 2"},
				{:name => "tag 3"},
				{:name => "tag 4"}
			]
		} # Or an ActiveRecord fetch
	end
	
	# views/articles/show.html.erb
	
	<%= Isotope.render_partial(Rails.root.join('app/views/articles/article.ejs'), :locals => { :item => @article }) %>
	
##### Client Side Example:

	# views/articles/show.html.erb
	
	<%= Isotope.render_template(Rails.root.join('app/views/articles/article.ejs'), :id => "article") %>
	

#### Sinatra

Actually the same usage, more or less.


---

Would love to hear your comments!

Elad Ossadon

* [http://devign.me](http://devign.me)
* [http://elad.ossadon.com](http://elad.ossadon.com)
* [elad@ossadon.com](mailto:elad@ossadon.com)
