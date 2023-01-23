## Rails Lite

We are going to create a very minimal Rails application that mimics the structure and behavior of a Rails app. 

We are going to follow the path that a HTTP Request would take through a Rails app, _creating components as needed_.


### Create the directory structure

* Rails keeps it's controllers and models in the app directory.  

```
mkdir -p app/controllers
mkdir -p app/models
```

* The initialization, config and route definition files are contained in the config directory.

```
mkdir config
```

* A collection of other ruby files are keep in the lib directory.  

	We are, _unlike in Rails_, create a lib/rails dir that will contain ruby that typically is provided by one of the rails gems.

```
mkdir lib
mkdir -p lib/rails
```


### Create the Rackup

Yep, this is how rails starts itself.  

```
touch config.ru
```

Add this to the config.ru  

```
 #\ -p 3000                                                                      
 # Above will Run this rackup on port 3000                                       

 # setup the environment                                                         
 require ::File.expand_path('../config/environment',  __FILE__)

 # Run the Rack app.                                                             
 run PersonApp::PeopleService.new
```

### Create the setup, initialization,code.

The config/environment.rb file will require all the files from directories.  

```
require 'pry'

 # Rails sets this global variable that points to the root directory
$RAILS_ROOT = "#{__FILE__.split('/')[0..-3].join('/')}"

 # Require all of the rails files in these directories                           
Dir["#{$RAILS_ROOT}/app/controllers/**/*.rb"].each { |f| require(f) }
Dir["#{$RAILS_ROOT}/app/models/**/*.rb"].each { |f| require(f) }
Dir["#{$RAILS_ROOT}/lib/**/*.rb"].each { |f| require(f) }

 # Require the file that defines the routes                                                            
require_relative './route.rb'

 # Require the Rack app                                                          
require_relative './application'

 # Create the People, power to the people, right on.                             
require_relative '../lib/seed_people'

```

The applicaton.rb file defines the Rack application. _Mucho importante_.

```
module PersonApp
  class PeopleService
    def call(env)
      request = Rack::Request.new(env)
      response = Rack::Response.new
      response_body = ""

      # Simple router                                                           
      # Will dispatch, call, Controller actions based on a URL path.
      response_body = Router.dispatch(request)
      response.write(response_body)
      response.finish
    end
  end
end
```

The config/route.rb file defines the __mappings__ between the URL path and the Rails controller and action.  
 _Remember that a controller is just a Ruby class. And an action is just a method_

```
module PersonApp
  Router.get('/people', "person#index")
  Router.get('/people/:id', "person#show")
end

```
#### Create a Person Controller.

A Controller will be called by the router to handle a HTTP request.  

In app/controller/persons_controller.rb.  

Note: The layout is a method in the ApplicationController. It wrap the HTML generated by the Controller actions.

```
module PersonApp

  class PersonController < ApplicationController

    def index
      @people = Person.all
      layout do
        contents = ''
        @people.each do |person|
          contents += render('html',  person)
        end
        contents
      end
    end

    def show
      @person = Person.find(params[:id])
      layout do
        render 'html', @person
      end
    end
  end

end
```

### Create an Application Controller class.

This is the base class of all of your application's controllers.  
In app/controllers/application_controller.rb.  

_We've hacked the layout here, typically the layout is NOT in the application controller_

```  
module PersonApp

  class ApplicationController
    attr_accessor :params

	# Wrap the contents returned from the block in 
	# standard HTML.    
    def layout(&block)
      contents = '<html><head><title>People App</title></head><body>'

      # invoke the block passed to this method
      contents += yield
      contents += '</body></html>'
    end

    def render(type, object)
      contents = ""
      if type == 'html'
        contents += object.to_html
      elsif type == 'json'
        # TODO: 
        contents += object.to_json
      else
        contents += object.to_html
      end
      contents
    end
  end
end

```

### Create the Person class.

This would typically be a Rails model, but we're using a Plain Ole Ruby Object (PORO) for now. In app/models/person.rb.  

```
module PersonApp
  class Person
    @@people = []

    def self.all
      @@people
    end
    def self.create(name, description, age)
      @@people << Person.new(name, description, age)
    end

    def self.find(index)
      @@people[index.to_i]
    end
    
    attr_accessor :name, :description, :agee

    def initialize(name, description, age)
      @name, @description, @age = name, description, age
    end

    def to_html
      "<dt>#{@name}</dt><dd>#{@description} is #{@age} years old</dd>"
    end
  end
end

```


### Create a Utils class.

This will provide assorted methods used in the app. Now need to know the implementation.

Create a lib/utils.rb file.   

```
module PersonApp
  class Utils
    def self.random_price
      (rand(1..10).to_f + rand(1..100).to_f/100).round(2)
    end

    # Extract the ID from the path into a params hash
    def self.extract_params(request)
      params = { }.merge(request.params)
      if request.path_info.include?(':')
        # TODO: fix, only good when at end of path
        id = path.split(':').last.to_i
        params.merge!({id: id})
      end
      params
    end
    
  end
end
```

#### Create a Router class.

This is an approximation of the much more complex router used in Rails. _No need to know the specifics of the implementation here_.

In lib/rails/router.rb  

```
module PersonApp
  class Router
    # A Hash of all the routes.
    # The key is the path from the URL
    # The value is a hash of the controller class and action name.
    # {controller: PersonController, action: action_name}
    @@routes = {
      'GET' => { },
      'POST' =>  { },
      'PUT' => { },
      'PATCH' => { },
      'DELETE' => { }                  
    }

    # Map get request to a path
    # path - /people
    # controller_action_str - 'persons#index'
    def self.get(path, controller_action_str)
      controller_name, action_name = controller_action_str.split('#')
      
      controller_name.capitalize! # TODO CamelCase
      controller_name += 'Controller'
       
      # Get the controller class from name, "PersonController"
      controller_klass = ::PersonApp.const_get(controller_name)

      # Add the mapping from the URL path to the controller and action.
      @@routes['GET'][path] = {controller: controller_klass, action: action_name }
    end

    def self.dispatch(request)
      # URL Path
      path = request.path_info
      # HTTP method
      http_method = request.request_method
      
      # See if path is in routes.
      controller_action_hash = @@routes[http_method][path]

      if !controller_action_hash
        # no path in routes, replace last segment with :id
        path = path.split('/')[0..-2].join('/') + '/:id'
        controller_action_hash = @@routes[http_method][path]
      end
      # The controller class
      controller_klass = controller_action_hash[:controller]
      # The action name
      action =  controller_action_hash[:action]
      # Create an instance of the controller
      controller_obj = controller_klass.new
      # Set the controller's params
      controller_obj.params = Utils.extract_params(request)
      # Call the controller action
      controller_obj.send(action.to_sym)
    end
  end
end

```

