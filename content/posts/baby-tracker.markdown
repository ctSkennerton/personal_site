---
title: 'Building a personal baby tracker with Pharo and Seaside'
date: Mon, 29 May 2023 12:35:00 -0700
tags: ['Pharo', 'Smalltalk']
---

One of the things that I've discovered about parenting is that
babies are tracked religously. How much are they sleeping? Are they
eating enough? Did they pee/poop the correct number of times in the
last 24 hours? This isn't just frettful new parent stuff (although
it totally is that too) but things that pediatricians want to know
and could be the early signs of problems when there isn't a good
way to know if there is a problem. There are many apps that will
help with this giving simple GUI to measure all sorts of things.
[Here](https://babytracker.info/) is a website that compares some,
and a quick search of the App Store will give many more. The problem
I have with all of these solutions is that you are minutely tracking
the first stages of your childs life and development and giving all
that data to someone else. The people who make these apps are running
a business and you're the product. Now I don't necessarily have an
issue with that, I'm using multiple apps that work by targeting ads
to me. I get it, but I didn't want to feed the personal data mining
machine. I didn't want to read a privacy policy or even worry about
it, I wanted something where there was no data to be given away and
sold. So I wrote [my own personal baby tracker](https://github.com/ctSkennerton/baby_tracker).

The requirements were pretty simple:

1. Dead easy UI so even in a sleep deprived state we could log updates.
2. Available across multiple devices.
3. Only accessible to people caring for our newborn.
4. Easy development and deployment. This is a personal project so it doesn't need to scale.
5. Have some fun.

I wrote an MVP that works for me using the Seaside framework with
[Pharo](https://pharo.org) 10.  This was my first time using Seaside
and I was really impressed with how easy it was to generate pages.
No need to learn a templating language like in Python or Ruby.
Instead, in Seaside, you write the HTML structure using smalltalk
code.  There was a big learning curve for me because of some initial
misconceptions about how Seaside works.  It's not just for creating
web pages, but for creating web apps. In Django/Flask or other
frameworks I've used the request contains all of the information
and state isn't saved. In Seaside, state is saved, I found that
thinking about Seaside to how something like React or Vue are used;
there is global state and this is saved for you between actions.
The difference being that Seaside acts server-side.
 
Seaside doesn't provide any persistence like Django or Ruby on
Rails. In that regard it's a bit more like Flask. Pharo has many
libraries for persistence I chose
[ReStore](https://github.com/rko281/ReStoreForPharo) as it looked
easy to set up. And it was! ReStore has a simple API that mimics
the collection API in Pharo but it also allowed for some interesting
customizations. ReStore has a simple way to add
translations for Smalltalk messages into SQL functions or expressions.
I was able to add in sqlite3's `unixepoch` function by using the
`asUnixTime` message that is understood by the `DateAndTime`:
```smalltalk
ReStore translateMessage: #asUnixTime toFunction: 'UNIXEPOCH(%1)' asSQLFunctionIntegerResult.
ReStore translateMessage: #asDate toFunction: ('date(%1)' asSQLFunctionWithResultClass: Date).

(BabyEvent storedInstances satisfying: [
    :each | each startTime > '2022-10-29' asDate ] )
    project: [
        :each |
        each type ||each startDate || each count || each durationInSeconds sum || each durationInSeconds average 
    ].
```
This kind of customization was increadably simple. I have no idea
how I could have done this in something like Django (or whether
such a thing is even possible).

One of the difficulties when using a more niche framework is the
lack of documentation for all of the use cases. One top of that
this was my first Seaside app, so trying to map my knowledge from
Django took a little time. For example, I couldn't figure out how
to keep track of the database connection, or where to store such
initialization information. After asking a few "stupid questions"
I was pointed to using the `WASession` class to hold "global"
information like the database connection. Here are a couple of links
I was pointed to:

* http://onsmalltalk.com/terse-guide-to-seaside
* https://edutec.citilab.eu/downloads/TFC-ASmalltalkByTheSeaside.pdf

Though I found that the [Reddit](https://github.com/svenvc/Reddit)
sample application for seaside is a great resource for looking at
the session and the deployment.

For deployment I found [this blog](https://nickjanetakis.com/blog/why-i-prefer-running-nginx-on-my-docker-host-instead-of-in-a-container) nice for not running nginx and I used Digital Ocean's tool
for creating the actual configuration (as well as their hosting to set up a small droplet)
