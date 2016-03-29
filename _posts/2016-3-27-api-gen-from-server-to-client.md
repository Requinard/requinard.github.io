---
layout: post
title: Swagger server and client generation
tags: software
---

Writing a client and a server is a lot fo work. Especially when starting from scratch. First you'll have to start out with writing a server. This involves writing your DDL (Data Definition Language). Then you have to deploy this, as well as sequences for your ID generation and a whole lot more.

And that's only working on the server. After this you'll have to write SQL statements for your clients, as well as a bunch of classes and related error handling to simply connect to the database. You'll also have to find some way to synchronize these SQL clauses to all applications that will use them.

And don't even think about changing your data schema after the fact. This will take hours to realize, as well as introduce a lot of bugs.

Outdated clients will also be a problem. People don't want to update and will keep using your old versions.

Now why do we deal with this mess? Because we kind of have to. But when you're starting a small project you don't want to deal with all of this stuff. You just want to get some work done.

That's where Swagger.IO comes in.

Swagger.IO is a standard to describe API's, plain and simple. It can describe Model Definitions, API Paths, but also authorization and a whole lot more.

Today we'll take a look at completely generating a working test server and accompanying client libraries to interact with it, without writing a single line of Java, Javascript or any regular language. All we'll use is the Swagger.IO definition and bash scripting.

These are the steps we'll be taking

### Steps

* Create a swagger file with definitions
* Use loopback to generate a server from data specification
* Use the swagger-codegen module to generate clients
* Do a quick test

### Requirements

- Java JDK (preferably 1.8 or up)
- NodeJS with NPM installed
- A text editor
- CLI interface
- Working internet

### Resources

- [Swagger.IO](http://swagger.io)
- [Loopback.io](http://loopback.io)
- [Swagger Codegen](https://github.com/swagger-api/swagger-codegen)


# Creating a swagger definition

At the heart of all of this autogen is the swagger definition. In our use case, we'll use this base swagger file simply as a data specification. You can add custom paths later if you want, but the basic model CRUD will all be auto-implemented.

Start out by creating a file called `swagger.yaml` (You can also define it as a JSON file, but we'll use YAML for this tutorial)

`
swagger: '2.0'
info:
  version: 0.0.0
  title: Swagger Tutorial 
host: localhost:3000
basePath: /
schemes:
  - http
consumes:
  - application/json
produces:
  - application/json
` 

This gives us the basic information of the API. The host is defined as `localhost:3000` since we will only be deploying this locally. If you want to deploy this to a server, change the host to your liking. Do keep the port though, it'll be important later.

Now with the basics setup we can start writing. First we'll need to describe a single API entrypoint so we can check for it being alive. Add this to the file

`
paths:
  /:
    get:
      responses:
        '200':
          description: OK
`

This describes a single API entrypoint at `/` that will simply return a HTTP 200 response.

Now that we have this entrypoint, we can start defining datamodels. I'll be defining only one.

`
definitions:
  TestModel:
    type: object
    properties:
      id:
        type: integer
      string_value:
        type: string
      date_created:
        type: string
        format: date-time
`

We now have a model, `TestModel`, that has 3 fields, an integer ID, a string value and a date_created.

You can define more models in this exact manner. This is all you need to start generating a server.

If you are on linux, you can use swagger-tools to validate your config file. Simply do the following

- `npm install -g swagger-tools`
- `swagger-tools validate swagger.yaml`

# Generating the server

So now we have a working swagger file that we want to turn into a server. First we'll need to install Loopback. Loopback is an api server with a lot of functions built in (Autogen from swagger, binding to database, auth).

run `npm install -g strongloop` to install Loopback itself. 

Run `slc loopback:app` to create a new server. Give it a fitting name. This is an interactive CLI client that will do most of the hard work. Set the directory to the same as the name and create an `api-server` from the preset options.

Now that we've got our server, we'll start generating the swagger definitions. Switch to the server directory and runn `slc loopback:swagger`. This will ask you for the location of a swagger file. Fill this in, and mark all the boxes on the models. Loopback will now generate the server itself.

Once that is done, simply run `node .` to serve the current directory as a node app, and navigate to `localhost:3000/explorer`.

## Only generating stubs

It is possible to only generate server stubs, and fill in the implementation yourself. Take a closer look at the codegen module.

## So what did we do?

We used Loopback to generate a server from scratch. We then generated data definitions from a swagger file. To add to this, loopback automatically added the various HTTP methods to make changes tot these models. We then ran the server on our localhost.

# Generating clients

Now we have a server running at `localhost:3000`. This server can already do all the basic IO. So now we need our client libraries.

We'll use a java application that will automatically generate adequate client libraries on build. Start a new java project with maven as your build tool, and add the following to your `pom.xml`:

[pom file](https://gist.github.com/Requinard/59edf1e94be2fbf5c40e)

As a source we'll use our loopback instance, which generated a full swagger document just for us. The swagger document is found at `localhost:3000/explorer/swagger.json`. Simply add this as a source and run `mvn clean compile`.

Boom. Swagger just completely generated your client libraries

# Using the client

Now simply add the following to your java file

`TestModelApi testModelApi = new TestModelApi(new ApiClient);`

You now have a working API. Simply run a get request on it to return all your data!

That's all there is to it

# After basic generation

After this basic generation, you can look into adding Authentication to your api, using API keys or oAuth, or you can start binding your datamodels to an actual database instead of the in-memory standard. For that, look into Loopback data sources. 

Good luck!
