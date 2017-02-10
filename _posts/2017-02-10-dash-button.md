---
layout: post
title: "A virtual dash button for your office"
date: 2017-02-10
categories:
description: ""
image: /assets/media/coffee-supply.jpg
image-sm: /assets/media/coffee-supply-sm.jpg
---
What is the worst thing that can happen in an office? No coffee!!

<small>Disclaimer: I am not a professional developer and do things like this in my free time to play around with code. I do not intend to aim for perfection but want to learn something. That helps me to make better decisions in my job as product manger, understand developers much better and for the most time is just fun to do.</small>

I read a post in our internal platform from a colleague. She wanted us to make sure our regular supplies are always in-stock (e.g. coffee or fruits). Like many of us she works from home ever now and then and needs to be informed  if something is running low. So I thought I should build something for that.

The idea is simple: Place an old tablet in the kitchen and show some buttons where you can tap any supply if it is running low or empty. If that is the case, drop her an email and send a message to our Slack.

![gif animation of the result](/assets/media/coffee-supply-animation.gif)

## API
The core of the backend is an API. Simply make a GET-Request to get the current status. E.g. something like this

```
  [
    {
      "id": "4e2a162865d9f466cc37ebe7bc001f7e",
      "value": {
        "name": "the last unicorn",
        "icon": "🦄",
        "status": "NOTMUCH",
        "lastUpdated": "2017-02-04 20:16:10"
      }
    },
    {
      "id": "4e2a162865d9f466cc37ebe7bc0023b5",
      "value": {
        "name": "coffee",
        "icon": "☕️",
        "status": "OK",
        "lastUpdated": "2017-02-04 20:31:28"
      }
    }
  ]
```

The module also has corresponding options to PUT, POST and DELTE an element and make sure data is current.
My API is written in Node.js using the [hapi framework](https://hapijs.com/) and packed into a simple docker container.

But wait, I said earlier that this should be a distributed solution. So we have to make sure the tablet in the kitchen is always up to date, even if the data is updated elsewhere. Good thing [socket.io](http://socket.io/) came to the rescue! Everytime changes are made to the supplies, an event is send to all connected clients to make them update their data. So no hard refresh needed.

## Frontend
To make it very easy and fast I just used the latest [Bootstrap](https://getbootstrap.com/) version (Alpha 4) and their grid options / media queries to not spend too much time on that part.

The most fun came with Vue.js. I built a component that sends requests to the API (using Axios, as [recommended](https://medium.com/the-vue-point/retiring-vue-resource-871a82880af4#.xkx3j9d6y)) and displays them using another component for the tiles. I really like the approach of capsulated components very much. I also found that Vue.js is not that hard to begin with (at least for such easy tasks). I started with their [guide](https://vuejs.org/v2/guide/) and also watched some screencasts on [laracasts](https://laracasts.com/series/learn-vue-2-step-by-step).

The design is very basic tough. Instead of adding an icon font or custom SVGs, I went for UTF-8 Icons. Simple, but beautiful.

And yeah, not to forget to include the socket.io client to make sure the data is updated whenever there is an update on the server.

There is also an admin page, that options for adding a new entry or deleting an old one. The same components for displaying the supply tiles is used as on the index page.

## Notifications
Notifications are handled quite simply. Again I set up two docker containers, one for my mailservice and one for slack notifications. Mails are handled via [Nodemailer](https://nodemailer.com/), for Slack Messages I made use of [slack-notify](https://www.npmjs.com/package/slack-notify)

## Database
As I needed some place to persist the data, I went for a CouchDB instance in a docker container. This might be a bit oversized and you could also use smaller solutions or a free SaaS product. But CouchDB is super easy to set up, does not need much resources and was the fastest and easiest way for me. The API has a decoupled module for communication with the database (currently using [nano](https://github.com/dscape/nano)), but that can easily be changed to anything else in the future.

## Webserver
Not much to say here. NGINX runs in a docker container with access to the frontend files. It delivers them on the root path and proxies all request to `/api/` to the API-container.

## Further improvements
The setting is simple and for sure not optimal. Things I definitely want to do soon are

- Use [webpack](https://webpack.github.io/) to enhance frontend performance
- Use something like [RabbitMQ](https://www.rabbitmq.com/) for communication between my modules. For now they all communicate with HTTP-Requests directly to each other, which is not a really nice if you want them to be as decoupled as you can get ;)
- An old iPad 2 is set up, currently looking for a stand or case to place it on the wall.

<small> photo credits: [by Brian Bilek](https://www.flickr.com/photos/khelvan/5516602973/) </small>
