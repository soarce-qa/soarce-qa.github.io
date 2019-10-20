# Overview
## Introduction

There are different scenarios for which this software offers a purpose. We aim to support our
fellow software developers, software engineers and software testers in every-day situations.
Keeping track of code execution across service borders is a bit more difficult. Also, when
encountering undocumented or "grown" code obfuscated by singletons, global variables and too
many public static methods, it is hard to detect what code is still in use and when it is
actually being used.

Using SOARCE in combination with either manual tests or even better an exhaustive Selenium,
Codeception or Katalon Test-Suite, you will be able to gather code coverage and a list of
involved classes and functions per use case per service. This allows for reverse search as
well, to answer questions like "If I change this function, what feature do I need to test?".
It can also be used to see what services are involved and what code inside them is executed
for a singular page load of the main application.

As of application version 0.13.0 and client version 0.7.0 we're even able to track requests
between services and draw a sequence diagram.

To be perfectly honest, this project was perceived out of necessity. After many years of
taking over one legacy software after the other and trying to make a sense out of them,
we took it upon us to finally create the tool, we would have wanted to have back then.

But let this be enough of an introduction, let's look at how this actually works:


## Theory of Operation

### An exemplary test run

Let's assume, everything is already set up - we'll cover that later. The gathering of
coverage and traces and the association with use cases will work almost automatically. For
most cases it is actually enough to tell SOARCE to start collecting and in addition to
tell it what the current use case is. An example selenium test for a login:

* request to SOARCE: start collecting
  * SOARCE forwards this command to all registered applications and services and returns
    only when all confirm success
* request to SOARCE: activate usecase "login"
* request to Application: load index page
  * Fill username and password
  * Send form
* request to Application: send credentials
  * Application calls an authentication service and asks if the credentials are valid
  * Returns an error to application
* assert an error message within application response
* Optional: request to SOARCE to stop/halt collection and then trigger some untraced
  cleanup requests
* Request to SOARCE: Start collection for the next use case
* ...


### How does it work?
#### Application

The application has a list of all services involved (see configuration). If you work
with docker-compose, both the SOARCE application and the applications and services to
be tested will have to have access to their respective networks.

Within the application there is set of Command and Control features that tell all the
linked services to start or stop collecting coverage and trace data. Furthermore the
application will know which use case is set to be active and treat all incoming coverage
or trace data as linked to said use case.


#### The Client / Plugin

We tried to develop this part as minimal invasive as possible. Currently the only thing
you will need inside your service containers will be xdebug. The client - including it's
dependencies - will be simply installed as composer packages, configured with a few
lines of JSON and will then handle everything automatically by intercepting calls to the
actual application and thus either execute certain tasks or run the coverage and tracing
automatically.

Code Coverage is sent back directly at the end of every request, trace information
however is more complicated. As we do not want to write hundreds of megabytes per request
to the harddrive just to read it once and then delete it again (even though SSDs might be
fast enough (actually not really ;)), we don't want to wear them out with that), we had
to become creative.

The client will create named fifo pipes within the container and then start a
(configurable) number of worker jobs that would each listen on one of the pipes. Tracing
information would be written to these pipes and the workers would read and analyze them
and send back the extracted information to the SOARCE application for storage. Redis is
used as a reliant provider for Mutex Locking as lock-files didn't work - at least with
php-fpm.

Redis is also used to store temporary request IDs for reverse lookup. This enables us to
track the request sequences without the need to pass on IDs between the services by hand.

#### Analysis

Data is written in a relatively raw form into a relational database. Through various
views and forms you can for example see the coverage of all tests in all applications
but are then able to narrow it down with the use of filters.

If you have an idea for another view which is not there yet, please let us know by
creating a ticket or even create a pull-request right away.


## Outlook

* Write data directly to redis as a buffer and only write single-threaded into database
* Solve large auto_increment values due to insert ignore side-effects
* Clients for different languages


## Contribution

### Free Software

This software is under MIT License. That means it is free of charge, you can do with it
whatever you want, we will not charge you for using it.

However, if you want to keep this project alive, there are a number of possibilities:
test it, contribute ideas for new features, contribute pull requests for new features,
write documentation, etc.

If you want to donate to support, I have an Amazon wishlist and a Steam wishlist. And as
this software is free as in "beer", we're always interested in novelty beer samples from
around the world.
