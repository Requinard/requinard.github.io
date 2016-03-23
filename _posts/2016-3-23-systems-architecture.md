---
layout: post
title: A high level architectural look at a system
tags: software,architecture
---

Large systems are always fun to design. There's a multitude of differing approaches an archtect can take towards designing the software beforehand.

I'll be taking a look at a project used in school, and I'll be desiging the system with differing architectural styles in mind, so that you can appreciate the choices I've made

## The Application

EmergenCIEM is an application that shall serve governments and larger instances. It is a system that can manage teams and emergencies as they come and go. It has to be extendable and easy to use.

The requirements are as follows:

- Manage incidents 
- Keep emergency teams up to date
- Keep multiple teams up to date
- Make it easy to manage multiple teams
- Prepare for disasters
- Analyze other sources to obtain relevant information

The following stuff should be basic:

- Be extendable
- Adapt to changes in an instant
- Run on multiple systems (Website, desktop, mobile)

## Design

There's a ton of ways to design applications like this. We'll start out with the one that most people will pick, Layer based.

### Layer based architecture

A layer based architecture divides the application into very specific components (Client, Server) and then uses layers to develop a working application. 

Let's look at an example client application

UI Layer <-> Business Logic Layer <-> Data layer <-> Data access layer <-> Database layer

In this example, a UI can only access parts of the business logic. It should not be able to directly touch the Data layer or execute any functions in there.

#### The good

Layer based is very good if you're building a monolith of software where requirements are clear.

#### The bad

It requires a lot of work to understand all of the code and it takes a while before you can deliver any working software

#### The ugly

A layer based application will quickly be stuck in a fashion. It's very hard to change parts of a layer architecture after you've committed to parts of it.

#### The verdict

Our applications scope is enormous. A layer based application would take a lot of work and effort, something that we cannot spare. The end-result is: Nope

### Event based architecture

So you've heard that this layer business is no good huh? Maybe we should implement the server like we do with a UI! Make everything respond to an event.

It sounds like a great idea, you only execute code in response to actual events happening.

Could it be too good to be true? Maybe. Let's look at an example first.

#### Case analysis

Let's take a bad case. A house is on fire and a call comes in asking for help. The ever-helpful attendant would then enter an alert into the system.

This is called an event. It will be put in a queue by a producer (Usually a keyval store like Redis) and wait for further examination.

This event will then be consumed by a consumer. This application will take an event from the event queue and execute code based upon it.

For example. In case of a fire, it automatically alerts the closest fire department to come put out the fire, but also call the neighbours and tell them to evacuate.

The great part is that the software can easily scale up horizontally, making sure that the application keeps working no matter what.

However, it also has a downside. Testing becomes nearly impossible since you will need a production cluster just to get started. The amount of moving parts is immense and will require a lot of sweat and oil.

All in all a good choice, but we can probably do better

### Microservices architecture

So the last one is kind of peculiar. My fellow second year students were a bit baffled. It's microservices. Instead of creating a monolith you create a bunch of services that try to work together as much as they can.

So instead of having a single client application, we have a bunch of them, a bunch of servers and a bunch of other tools running. All independent and unrelated. Sounds scary right? Let's look at an example.

#### Services defined

These are all seperate applications

- API Service
- Desktop client
- Mobile client
- Website
- Social Media Monitoring
- Notification Service
- AV Streaming Server

So how does all of this help? We'll start with the central piece, the API Service. In it's first incarnation it will be a dumb datastore that applications can request data from.

This will take care of all database IO and will start handling Api requests and events.

This means that our clients are usually only an interface to our API (Until we start developing the streaming servers) and we don't have to write all the SQL code a bunch of times.

But we do now have a bunch smaller applications, each with a very specific set of goals it needs to achieve. they are also only loosely coupled to eachother. The only actually required piece is the API Server.

Not only that, but the code is very isolated and does not duplicate as much. It can also easily scale up to meet more demands, since there is no tight coupling of components.

## Conclusion

The end-decision was pretty easy. Layers was a no-go for an agile application. Events was too much work for our small team (4 man, including me). Microservices seems like the perfect choice for an application this size and with a team this size.

Now, only time will tell whether we succeed. Let's see in 15 weeks!