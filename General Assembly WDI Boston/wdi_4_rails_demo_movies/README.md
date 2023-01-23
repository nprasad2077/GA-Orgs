![General Assembly Logo](http://i.imgur.com/ke8USTq.png)

## Objectives
* Create a simple front-end app that will use Ajax to get responses provided by a simple HTTP server.
* Use Ajax to get a list of movies from this simple HTTP Server.
* Draw Diagrams that show the flow of a HTTP Requests and Responses.
* Use Ajax to get a list of movies from a **pre-existing** simple Rails application.
* Create a Rails application that will provide this list of movies.
* Access a list of movies and songs from this Rails API using HTTP clients, Browser and curl.
* Access a specific movie and a specific song.
* Draw Diagrams that show the flow of a HTTP Requests and Responses.


## HTTP client/server diagram.

![Rails HTTP Client Server](http_client_server.png)

Yes, the client is the browser. And yes the server is Rails.

[HTTP GET and POST](https://www.youtube.com/watch?v=kGOpY2J31pI)

[Very longish HTTP Lesson](https://github.com/ga-wdi-boston/wdi_2_http_demo_overview)


## Create a frontend app

**Create a directory for a front-end application.**

```
mkdir movies_frontend
cd movies_frontend
touch index.html
touch movies.js
touch movies.json
```

**In the index.html add**

```html
<html>
<head>
  <title>Movies</title>
</head>
<body>
  <h1>Movies</h1>
  <ul id='movies'>
  </ul>
  <button id='movies_button'>Get Movies</button>
  
  <script type="text/javascript" src="http://code.jquery.com/jquery-2.1.4.min.js"></script>
  <script type="text/javascript" src='movies.js'></script>
</body>
</html>
```

**In the movies.js add**

```javascript
$(document).ready(function(){
  var $moviesList = $('#movies');

  // Get the movies from the movies.json file served from
  // the web server listening on port 5000.
  var movies_url = 'http://localhost:5000/movies.json';

  $('#movies_button').on('click', function(event){
    $.ajax({
      url: movies_url,
      dataType: 'json'
    })
      .done(function(movies_json){
        var movies = JSON.parse(movies_json);
        movies.forEach(function(movie){
          $moviesList.append("<li>" + movie.name + "</li>");
        });
      })
      .fail(function(data){
        var errorMsg = 'Error: Accessing the URL' + movies_url;
        alert(errorMsg);
        console.log(errorMsg);
      });
  })
});

```

**In the movies.json file add.**

```json
"[{\"name\":\"Affliction\",\"rating\":\"R\",\"desc\":\"Little Dark\",\"length\":123},{\"name\":\"Mad Max\",\"rating\":\"R\",\"desc\":\"Fun, action\",\"length\":154},{\"name\":\"Rushmore\",\"rating\":\"PG-13\",\"desc\":\"Quirky humor\",\"length\":105}]"

```
**Run a SIMPLE web server**

This will run a very simple web server on port 5000 of your local machine. 

```
ruby -run -e httpd . -p5000 
```

The web server we're using here is provided by Ruby. It's call WEBRick. It's OK for development on your local machine but should not be used in production.


**Get the frontend app**

Go to `http://localhost:5000` to get the frontend app.

This will:

1. Send a HTTP Request from the browser to get the index.html from the server.
2. Send another HTTP Request from the browser to get the jquery library.
3. Send a HTTP Request to get the movies.js file.
4. Run the contents of the movies.js file after the DOM is loaded.

## Lab: Draw a Diagram explaining the flow of HTTP Requests/Responses.

* Don't forget to show the HTTP Request/Response generated by Ajax.

##Let's Use an existing Rails application

That provides us with a Movies API.

Follow the instructions in this repo.
[Simple Rails Movies API](https://github.com/ga-wdi-boston/simple_rails_movies_api)

## Let's use this Rails Server/API for the frontend app.

**Change the movies.js file**

Ajax request must now use the URL `http://localhost:3000/movies'

And we should not need to explicity parse the response back from the Rails server/api.

```javascript
...
 var movies_url = 'http://localhost:3000/movies';
 
 ...
 
	 .done(function(movies){
        movies.forEach(function(movie){
          $moviesList.append("<li>" + movie.name + "</li>");
        });
      })
```
## Let's Create a new Rails application.

This will be the same Movies API that we used above.

**Install Rails, yep it's just a ruby gem.**

*Note: this can be done from within any directory*  

```
gem install rails
```  
**Create a Rails application named movies_app.** 

```
rails new movies_app -T --database=postgresql
```    

**Review Rail's directory structure.** 

Change into this movies_app directory and take a look at how Rails gives you a very clear location to place all the code you'll be writing. 


```
cd movies_app
subl .

```

No worries about what all this means, we'll get it over time.

**Start up rails.**  

```
rails server
```

**Access the default Rails URL.**

In your browser, go to *http://localhost:3000*

You should see this error. 
>> ActiveRecord::NoDatabaseError  
>> FATAL: database "movies_app_development" does not exist  

Oh,no Databases. We're going to see more about DB's later.


**Create a database for this rails app. *Rails always need a DB*.**

And Restart rails.

```
rake db:create

rails server
```

We are going to see more about how Rails works with a database later. *Hand wave, Hand wave*

**Access the default Rails URL.**

In your browser, go to port 3000.

Ya, you should see the Welcome Aboard page. Rails is running!!!


**Take a look at all your movies*

Go to *http://localhost:3000/movies*

Bummer, you get a routing error, Boo Hoo, waah waaa. Let's fix it.

But, lets talk about **routes** first. 

### Routes. Create a Route for URL path '/movies'

Rails routes determine what will happen when Rails sees a specific URL path and a HTTP Request.

We send a HTTP GET request to a specific URL. In the case above are we trying to send a HTTP GET request to '/movies'.

But, we have NOT told Rails what to do when we go to this URL path?  
*Lets fix that.*

**Open the file config/routes.rb and add the below.**  

```
get 'movies', to: 'movies#index'
```

We are creating a Rails route that will say to Rails.

When you see the URL path '/movies' invoke the index method on the class MoviesController.  

**Access the default Rails URL.**

*Go to *http://localhost:3000/movies*  

You should get the error *uninitialized constant MoviesController*

OK, whats going on here is that Rails saw the /movies URL and the ``get 'movies', to: 'movies#index'`` route said **when you see this /movies URL go to the MoviesController class and invoke the index method**

But there is no MoviesController class or index method?

*Let's create it after talking a bit about Controllers*

### Controllers and Actions: 

**Create a Movies Controller with a index method/action.**

Add this to the file app/controllers/movies_controller.rb.  


```ruby
class MoviesController < ApplicationController
  def index
    render :text => "All my movies"
  end
end
```

Lets talk about what's going on here.

### HTTP client/server diagram with Rails.

Lets draw a diagram for whats going on here!

## Lab: Create a Rails Songs App.

Create a rails app that will respond to *http://localhost:3000/songs*

* Create a rails app, like above, that will be called songs_app.  
* Create a route for */songs*  
* Create a songs controller.  
* Create a index action.  
* Render 'All my songs'
* Access this at http://localhost:3000/songs.

* Draw out a diagram of the flow of the HTTP Request and Response from the client/browser to the Rails server. 
	* Show the browser.  
	* Show the HTTP Request.  
	* Show the router.  
	* Show the controller.  
	* Show the HTTP Response.

## HTTP Request for JSON.
**Create a Movies Controller that will return JSON representation of a list of movies.**  

```ruby
class MoviesController < ApplicationController
  # TEMPORARY ONLY!!
  def movies 
    [
      {name: 'Affliction', rating: 'R', desc: 'Little Dark', length: 123},
      {name: 'Mad Max', rating: 'R', desc: 'Fun, action', length: 154},
      {name: 'Rushmore', rating: 'PG-13', desc: 'Quirky humor', length: 105}
    ]
  end 

  # GET /movies
  def index
    render :json => movies.to_json
  end
end
```

## Lab: Provide a JSON Representation of Songs.

* Create a method, named songs, that will return an array of hashes. Each hash will represent one song.  
* Each song will have a title, duration, price and artist name.
 
### Setup Rails to allow Cross Browser HTTP Requests**

*This is magic, for now. Trust me, it'll make sense later.*


**Add this line to the Gemfile**

```
gem 'rack-cors', :require => 'rack/cors'
```

**Install this gem in rails**

```
bundle install
```

**Add these lines to the config/application.rb**

```
config.middleware.use Rack::Cors do
  allow do
    origins '*'
    resource '*', headers: :any, methods: [:get, :post, :patch, :put, :delete, :options]
  end
end
```

**Restart Rails**

```
rails server
```
