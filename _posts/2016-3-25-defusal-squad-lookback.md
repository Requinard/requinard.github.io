---
layout: post
title: Defusal Squad, a projects retrospective 
tags: software
---

Looking back at projects is always a fun endeavour. Mostly because the 20/20 hindsight can teach you so much about software engineering itself. I want to see what leason I can take from my project, Defusal Squad

You can find the project itself [here](https://github.com/Requinard/TeamTab)

## What is this 'Defusal Squad'?

Defusal Squad is the name we gave our multiplayer co-operative/competetive game. It features heavy teamwork and communication, but in teams against eachother. The game itself was inspired by Spaceteam and is mostly our own java adaptation of this.

It's built on JavaFX, featuring our own networking stack. A fun little exercise.

## Application Design

Defusal Squad's design was made with the intent of it being fully distributed. People should be able to play it only using a local network connection. This prompted a design decision we had to make.

Our clients would be multi-purposed. A single application could be a client as well as a server. It had to be able to switch modes on the fly. While this was a big implication, we only experienced minimal fallout because of this.

## Setting everything up

We layered our application in 4 distinct layers. The GUI, the game logic, the networking mediation and the networking itself. We'd run a total of 4 threads.

- JavaFX UI logic
- Host mediator
- Client mediator
- Networking

The UI was set up using a View-Controller pattern, with a view doing the rendering and a controller handling all the clicks.

The game itself was defined by an interface, which allowed us to subclass hosts and clients. Clients would simply hand data from UI to networking, while hosts mostly responded to incoming requests.

A host would run 2 versions of the game. A host to manage it, and a client to actually play with.

Our networking was set jup in a way that we'd use a 3 layered approach to networking itself. At the base we have a layer that reads string inputs and outputs. On top of this we built the `NetworkMessage` class, which stored data related to the message and some metadata (Sender and receiver).

On top of this we have a `NetworkRequest` class. This handles the actual logic. A message was always constructed in a very specific manner, which a `NetworkRequest` could handle.

A `NetworkRequest` simply exists as a method (GET, POST or STATUS), a path (`/users/`) and a message.

## Bugs that took us forever

There's been a couple of bugs that really annoyed us and took some time to find. Here's a couple of them

### Handrolling Networking

Handrolling our networking was fun. We managed to construct an actual protocol on top of TCP so that we could get our data across. The only downside was that our hosts and clients were both polling based. They'd ask our `NetworkServer` for a message, and would try and handle it if it could.

This was in contrast to event based networking, where a network package would run it's own code in response to being received.

### GSON never executes constructors

A whole set of bugs had to do with GSON. When a received GSON object is cast to the object it's representing, the code's constructor is never executed. This managed to bypass failsafes we had against Null values.

This one took us a while, since the code in question was being run in it's own thread. An uncaught exception in a seperate thread will result in the thread silently dissapearing. Tip for the future: build null-checks for all the objects you are sending over the wire with GSON.

### Losing packages

Running a host and client at the same time is great fun. We managed it, but started dealing with package losses only for the game host. As it turns out both our mediators were pulling packages from the same queue, resulting in a race condition. Since a client would only handle `STATUS` and a host would handle `GET` and `POST` packages.

When a client would pull a `POST` package, it would find it unusable, and discard the packages.

We managed to fix this problem by simply switching to a priority queue for our networking application. A requeued request would be added to the queue with high priority, so the host thread would pull the system.

Adding a `Thread.yield()` managed to fix all the problems we were having with package loss

## What we would do different next time

- We'd use event based networking instead of polling based networking
- Manually write the JSOn serialization modules to stop bugs from happening
- Implement a UI with loading screens

## Conclusions

the project went reasonably well. We managed to deliver a software package that all systems could run (We tested on windows, mac and linux) and that would run with only a local connection. I'd call this project a success!
