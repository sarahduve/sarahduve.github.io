---
layout: post
title: "Don't Forget to Float"
date: 2013-06-27 17:39
comments: true
categories: 
---

So here's a riddle for you. When do New York's 300+ Citibike stations look like this?
![a funny map](/images/float_map.png "Something's wrong here.")
Answer: The same time 5 / 2 = 2 (i.e. when you're doing integer math in Ruby).

But allow me to step back a bit. The above map arose during the hacking portion of the awesome Citibike Civic Hacknight held last night by [BetaNYC](http://www.meetup.com/betanyc/) (and co-hosted by Flatiron TA [Ashley](https://twitter.com/ag_dubs)!). While Citibike has yet to release a public API (though I'm told that's imminent), developers have been able to expose a JSON [API endpoint](http://citibikenyc.com/stations/json) in Citibike's source code that gives a live snapshot of the status at all stations. By continually scraping this data, some people have been able to put together [some](http://sites.google.com/site/citibikestats/) [pretty](http://experimenting.alastair.is/citibike/) [cool](http://data.citibik.es/) [things](http://citibikedata.com/), many of which were presented last night. One of the highlights for me came not from a hacker or data scientist, but from Anthony Townsend, a research fellow at NYU's Rudin Center. Anthony explored the possibility of redesigning neighborhoods around Citibike, using it to effectively spread the value of the subway over a larger area, and thus drawing New Yorkers into farther-flung corners of the city. You can view his deck [here](http://www.slideshare.net/anthonymobile/bike-hack-night-townsend).

Back to the hacking!

Ashley was kind enough to put together a little mini lab for the small group of Flatiron students that attended. We used the [MultiJSON](https://rubygems.org/gems/multi_json) gem to parse a snapshot of the station feed, turning it into a Ruby hash for us to play with. Since we were just getting started with Citibike data and didn't have weeks of scrapes to work with, plotting the stations on a map seemed like a good simple exercise to start with. We set to work implementing that with [Leaflet](http://leafletjs.com/) but quickly realized that on its way to becoming a Ruby hash, our JSON data had undergone a few changes. Namely, our GPS coordinates, which previously had eight decimal places, were now eight digit integers. 

<script src="https://gist.github.com/sarahduve/92415a43a74b9092339c.js"></script>

<script src="https://gist.github.com/sarahduve/4392058dba9dae6ab812.js"></script>


Okay, no problem - easy fix, right?
<script src="https://gist.github.com/sarahduve/2bfc53469d82d6ebe9b0.js"></script>
Not so fast. For those of you who have already figured out the problem here, please bear with me because I'm about to explain the meandering route I took to get there.

Part of the problem with our code at this point was that it made it look like only two sets of our coordinates were being plotted, when we should have been iterating over more than 300. I spent a lot of time staring at this, trying to figure out what else could have potentially gotten lost in the JSON translation, and what other data points these markers could inadvertently be representing. Some key names underwent some minor changes (e.g. "latitude" to "lat"), but otherwise nothing stood out as too glaring, so I decided to take a closer look at the two actual locations.




<iframe width="375" height="300" frameborder="20" scrolling="no" marginheight="10" marginwidth="10" src="https://maps.google.com/maps?f=q&amp;source=s_q&amp;hl=en&amp;geocode=&amp;q=40,+-75&amp;aq=&amp;sll=40,-74&amp;sspn=0.128081,0.32341&amp;ie=UTF8&amp;t=m&amp;z=14&amp;ll=40,-75&amp;output=embed"></iframe><br /><small><a href="https://maps.google.com/maps?f=q&amp;source=embed&amp;hl=en&amp;geocode=&amp;q=40,+-75&amp;aq=&amp;sll=40,-74&amp;sspn=0.128081,0.32341&amp;ie=UTF8&amp;t=m&amp;z=14&amp;ll=40,-75" style="color:#0000FF;text-align:right"></a></small> 

<iframe width="375" height="300" frameborder="10" scrolling="no" marginheight="0" marginwidth="0" src="https://maps.google.com/maps?f=q&amp;source=s_q&amp;hl=en&amp;geocode=&amp;q=40,+-74&amp;aq=&amp;sll=40.697488,-73.979681&amp;sspn=0.507051,1.29364&amp;ie=UTF8&amp;t=m&amp;z=12&amp;ll=40,-74&amp;output=embed"></iframe><br /><small><a href="https://maps.google.com/maps?f=q&amp;source=embed&amp;hl=en&amp;geocode=&amp;q=40,+-74&amp;aq=&amp;sll=40.697488,-73.979681&amp;sspn=0.507051,1.29364&amp;ie=UTF8&amp;t=m&amp;z=12&amp;ll=40,-74" style="color:#0000FF;text-align:left"></a></small>

Hmmm, so the one thing these seem to have in common is their perfectly rounded coordinates...perfectly rounded...hmmm...this sounds...familiar.

Time to do something I should have done a long time ago.

![integer math](/images/integer.png "That's not right!") 

Ah ha! 

If you use integers in Ruby arithmetic, you get integer answers. As it turns out, those "two markers" were actually all 300+ stations, directly on top of each other. When rounded, all became either 40, -75 or 40, -74.  If you want floats (i.e. decimals), you need to use them in the operation whether by adding decimal places yourself or converting with `.to_f`.

![float](/images/float.png "There we go!") 

Problem solved.
![a real map](/images/real_map.png "That's more like it")

This is some very basic Ruby math that we all learn early on, but can easily forget because we generally don't have much reason to use floats in our programs (40.742354 artists in your playlist, anyone?). And it's not often that we get such a cool visualization of this common error, so I'm kind of glad this happened. Between the maps and this post, here's betting I don't make it again!

