---
layout: post
title: "Tech debt demotivates and can kill your pet projects"
date: 2016-02-01 01:26:30 -0200
comments: true
draft: true
categories: tech
---

I've been working on a side project named My Assets (Meus Ativos in Portuguese), this is what I learnt about tech debt.

<!-- more -->

# The Android App

## A brief story

It all started as a coding exercise, at the time my brother and I were studying mobile development. I wanted to build a full CRUD app with customised lists and some simple report. So we started with a CRUD for investment operations, read, a transaction tracker.

As we started we thought the idea was good, we knew at the time a couple of people who would use this app for sure. Most of our friends manage their investments using Excel spreadsheets. The more progress we had, the more excitement and of course it was our new pet project.

The app was so simple, the database was a sqlite file running on the phone (no cloud, no backups). There was no need for creating user account or anything like that. We published on Google Play and people simply liked it. More and more ideas and suggestion were coming up.

### First lessons here

Please, always ditch your prototypes. The main reason is not code quality, it's not honour but it's anchor bias. Once you build a functional prototype, you may have the idea to refactor it. But anchor bias will keep you on the same architectural, same flow and same problems.

## The first crisis

In six months of development our productivity has dropped significantly. I remember feeling like a snail, but what was wrong? It was  two developers, writing code with over 70% of unit test coverage, how that was so unhealthy? Isn't that enough?

We found at the time that:

  1. The folder structure was a standard MVC: ./controllers ./models and ./views. You know what I'm talking about.
    That is a big problem in the long run, because you never think of this, "where is my controllers? Ahh of course they are in controllers folder.".
We never search for the wires. 90% of the time we look for the business, "There is a bug on page X. The question is: Where are the files related to that page?"

  2. Android's activity class has too many dependencies, it's too hard to test. Our presentation rules were not tests at all, and the question was do we wanted to test it?

  3. If the project continues to grow like this, what is going to happen to the project structure? Can we create modules?

## The big refactoring

The solution was the MVP (Model View Presentation) architecture. The untested presentation rule would live in an isolated class. Controllers would delegate all the business to interactors. Models, well they are as skinny as possible.

The second step was to rename the folder structure to represent the App itself. For each page screen we created a folder with its feature inside in sub package. 

It was not an easy job, we had to hold new features and refactor the architecture. One or two weeks later it was all done and we were proud of that.

### Lesson learned

The project started with a poor architecture since it's prototype. Everything is much harder to change once you have users, they get used to the app as it is.

Always aim for an architecture that supports growing. Think about what if I have 100 controllers? What if I have 100 features? What if I get to a 100 models?

# Web version

We knew that storing data on the user's phone wasn't a good idea. A week before the first release, we introduced a log mechanism inspired on git. It  was keeping track of all data manipulation made by the user. It allowed us to recreate the same database state later, on the server side. 

One year least, we had over 200 downloads. I had moved to another town quite far. I joined ThoughtWorks and time was an issue. I realised that I was learning more about Ruby On Rails there, and I wanted to keep practicing it! There came the idea to build a website:

  1. So we would have a landing page, to present our app.
  2. We could have a web version of the app, it would allow us to do better reports.
  3. Cloud storage for the user's data.

Again, I remember starting with no big plans or intentions. I wanted to do a good kick start, show it to my brother and then we would figure what we could do. It started as a very simple landing page based on a free bootstrap template. Added some screen shots, free pictures I found on internet, it wasn't good whatsoever!

He liked the idea and we kept moving on. So while my brother was improving the landing page I was build the storage. Ruby on Rails productivity was insane.

Once we got a basic landing page we published it, with a link for Google play. Also we publish the endpoints to that would allow the mobile app to synchronise data with the cloud.

And later re implemented all the features from the app on our website. We got duplicated business rules in two different platforms.

### So... where this is going?

Well drawing a conclusion here. Again, we did the same mistakes again. Haven't planned much before starting. Accumulated technical debts from the day one.

It's true that we still manage to get the App and Website out, and currently we have more than 10.000 users. But everything could be done smarter and better.

When it comes to a side project, you need productivity to keep motivated. Having a nice code that you enjoy to work on. If you build a mess, your passion will fleet.

Start small, build prototypes then throw them away. Try to think ahead and solve the puzzle one piece at the time.
