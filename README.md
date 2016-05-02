Rails 5 API app & Ember
=======================

I'm learning rails 5 and ember, so I'm putting together this guide as a form to track what I'm learning and keep
progress. Some of my information sources are:

*   http://edgeguides.rubyonrails.org/api_app.html

I build a rails 5 api app, and hook up that to an ember frontend.
The goal is to build the Todo app (a popular choice).

### What is an API Application?

Traditionally, when people said that they used Rails as an "API", they meant providing a programmatically accessible
API alongside their web application. With the advent of client-side frameworks, more developers are using Rails to
build a back-end that is shared between their web application and other native applications.

For example, Twitter uses its public API in its web application, which is built as a static site that consumes JSON
resources.

Instead of using Rails to generate HTML that communicates with the server through forms and links, many developers are
treating their web application as just an API client delivered as HTML with JavaScript that consumes a JSON API.

TLDR; Many parts of Rails are still applicable even without the view layer.

Backend
-------------

### Generate the Rails API only application

    $ rails -v
    Rails 5.0.0.beta3
    $ rails new /tmp/my_api_app --api

This will do three main things for you:

*   Configure your application to start with a more limited set of middleware than normal. Specifically, it will not
    include any middleware primarily useful for browser applications (like cookies support) by default.

*   Make ApplicationController inherit from ActionController::API instead of ActionController::Base. As with
    middleware, this will leave out any Action Controller modules that provide functionalities primarily used by
    browser applications.

*   Configure the generators to skip generating views, helpers and assets when you generate a new resource.

See [Changing an existing application](http://edgeguides.rubyonrails.org/api_app.html).

### Scaffolding the Todo resource

The Todo items in the Ember application have two attributes:

*   a string title and
*   a boolean isCompleted.

The following step to build our backend application is precisely to add a _resource_ representing these Todo items.

    rails generate scaffold todo title isCompleted:boolean

A scaffold in Rails is a full set of model, database migration for that model, controller to manipulate it, views to
view and manipulate the data, and a test suite for each of the above.

The generator sets up necessary components to route for the resource. Now update the database with the change.

    rake db:migrate

### Choose the appropriate JSON serialization format

Our Rails API only application is going to respond incoming requests in a given JSON format. The process to convert the
data into this format is called _serialization_ and this will be possible in our backend application thanks to Active
Model Serializer adapters.

Start the web server.

    rails s

And create a Todo using curl

    curl -H "Content-Type:application/json; charset=utf-8" -d '{"todo": {"title":"Todo 1","isCompleted":false}}' http://localhost:3000/todos

Using the default json adapter, a Todo item would be serialized like:

    {
      "id": 1,
      "title": "Todo 1",
      "isCompleted": false
    }

But we need to pick a JSON format matching our Ember application, selecting a format that works well with the Ember’s
RESTAdapter.  The main requirement specified by the RESTAdapter is the presence of the _root object’s key_ as part of
the JSON payload, as it is explained in the Ember RESTAdapter documentation.

It means we want to serialize a Todo item like this:

  {
    "todo": {
      "id": 8,
      "title": "Todo 1",
      "isCompleted": false
    }
  }

This can be done with Active Model Serializer.

1.  Add `gem 'active_model_serializers', '~> 0.10.0.rc1'` to the Gemfile.
2.  In `./config/initializers/ams.rb`, add `ActiveModel::Serializer.config.adapter = :json`.
3.  Add `include ActionController::Serialization` to `app/controllers/application_controller.rb` like this:

    class ApplicationController < ActionController::API
    +    include ActionController::Serialization

