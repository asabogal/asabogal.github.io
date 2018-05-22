---
layout: post
title:      "My first RUBY CLI-GEM Project"
date:       2018-05-22 15:06:58 -0400
permalink:  my_first_ruby_cli-gem_project
---

Last night I finished my first Flatiron School project. It’s a Command Line Interface (CLI) Ruby Gem. 
It was a challenging project, but more than that, it was a fun project! 

This was the first time I started working on creating an application from scratch. That is, working with a blank canvas with no tests to pass, no environment setup, no spec files, nothing. Until then, I had been working with the Learn IDE application that emulates the Atom Text Editor. The IDE runs from the school’s servers so all the labs came pre-loaded with the necessary gems and dependencies needed for the program to run.

So since I would be building the whole project from scratch, I decided to switch from the IDE environment into a local environment. After all, this is the way “real” programmers work right? I was hesitant because now not only did I had to figure out how to complete the project, but also had to learn how to work with a local environment. I took the leap anyway and man, I couldn’t be happier! It does feel like you are working the right way now. Setting up the environment took about an hour or so, I did some additional tweaking to my system and Terminal, but the process is really easy. If you’re on a Mac you can find the information here: http://help.learn.co/workflow-tips/local-environment/mac-osx-manual-environment-set-up

The first challenge was to setup the file structure for the gem. When you have Ruby installed in your system, you can run `bundle gem <project_name>` and it will create the file structure you need. I suggest watching Avi’s “Daily Deals” project walkthrough where he explains this process in detail and also elaborates on how to approach the project in general. 

After setting up the structure, I thought about my idea and how to implement it. I decided to create an application that allows a user to browse a list of credit card offers, select the reward type he/she is interested in, and get information on the best credit card for that reward type. 

So what would I need for this? A user interface, a scraper class to extract the data online, and a card class responsible for creating the objects.

I first designed the user interface through a CLI file. This would be responsible for the user and program interactions. A good idea is to sudo code a draft in the CLI file that will later allow you to put “real” code into it. I think it makes easier for you to think of the overall functionality of the program and how different classes will interact if you do this before you start the actual coding.

Once my CLI logic was set, I drafted a Card class that will hold the attributes of a Credit Card i.e. Apr, Annual Fee, etc… I did a basic setup for the class. `#initialize, #save, #display_all`. I thought to keep this class as simple as possible so that the functionality will be easier to pass to the CLI.

Now I needed to collect my data. I struggled here. The site I chose had different containers for different data. That meant I couldn’t just get all the data from one single iteration or page scrap. I thought about scraping the different containers, make the necessary variables for each attribute and store those attributes as key:value pairs in a shared hash.
Then initialize cards with the hash elements. I played around with this idea and most of it worked, but I foresaw it getting messy quickly. I wanted simplicity. I eventually found a way to scrape two deferent containers and iterate over their data at the same time. The trick? .`each_with_index`. Oh yes! You can iterate over two arrays if you use this method. 

In my situation I had a Nokigiri array with all the card reward types and a second with the main card attributes.
Since they had the same amount of elements, I set the reward return of each iteration to the index of the iteration:
`reward = reward_array[i]`. 

And that was it! My main problem was solved. The rest of the time was spent tweaking the user interface until it was presented the way I wanted.

I’m really happy with the results because I achieved exactly what I had in mind. Also, I learned a ton! What I take away the most from the project is how to work without the need of a test driven development tool, and instead use the terminal and console to troubleshoot. Tools like `binding.pry` are life savers and your best friend!

You can check my gem here: 

[https://github.com/asabogal/top-credit-cards-CLI-gem.git](http://)
