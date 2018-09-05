---
layout: post
title:      "DJ CHARTS a Sinatra App"
date:       2018-09-05 00:45:04 +0000
permalink:  dj_charts_a_sinatra_app
---


I just finished my Sinatra section project. I must say I had a lot of fun working on it.

The idea was to create an application that will allow a DJ like me keep track of his/her record charts. A record chart is basically a top 10 or top 5 list if your favorite music records/tracks or the top records/tracks a DJ has been playing out.

It was the first time that I worked on an interactive app that was user friendly, meaning that the user has more interaction with it than just entering yes or no, or a predetermined set or responses on a command line. This also made it challenging.

I didn’t spent too much time on the front end design of it. I managed however to make it “easy on the eyes” but would’ve loved to have the time to further style the layout and pages with bootstrap. 
Most of the time was spent on user authentication, validation and correct redirection after user actions. One issue I had was that I didn’t validated the uniqueness of chart names. I didn’t wanted to do this as a user should have the liberty to name their chart as they want. specially when is very common that a DJ names a chart by the month when it was charted, i.e. September, or January Chart, etc… This issue became apparent towards the end of the project when I had several registered users and charts, and some charts where being redirect to other chart pages. This was because I was identifying charts by their slug. The fix was to change all the paths and include the chart id as the main identifier, and change all the search methods to “search by id” instead. The result looked something like this: `…/charts/7/chart-name` I felt leaving the slug was important specially when these things are usually identified by their name by the user. 


Another issue I faced was creating `add record` button and path. This function is only available inside a chart show page, so I had to figure out how to send the correct parameters to the `records/new` path. I used a `get` method form with a hidden action which value was the attribute/parameter I need for the new record page to be able to create the record, identify the chart it belongs to and authenticate the process. This felt really good when I discovered this as it was done so by pure programmatic thinking and logic of my own! 



