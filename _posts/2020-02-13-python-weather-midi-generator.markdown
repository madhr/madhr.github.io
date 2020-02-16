---
layout: post
title:  "Song generator based on weather forecast"
abstract: "Python script that creates midi songs based on weather forecast for given location."
date:   2020-02-13 21:37:32 +0100
categories: python midi weather generator
---

Writing music, huh? Why would anyone do it manually, when we have computers?

Well ok, I like to do it manually and millions of people do it manually as it's a wonderful thing. But what if we had a script that would create the music for us? Interesting idea, but it means that we need to find something that will give us some input data on which the script's calculations will be based. That's how I came up with the idea of using weather forecast as an input data. 

Remember these classical composers who wrote music because they got inspired by weather, like Vivaldi and his <i>Four Seasons</i> or Chopin's <i>'Raindrop' Prelude Op. 28 No. 15</i>. Absolute masterpieces, but it's 2020 now and we have technology, so I'll try to write code that will write music for me.

Here you can check out one of the results of my script:

<iframe width="100%" height="150" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/761138425&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe>
<br>
It's a weather song for US town called Adak. I picked that one, as on the day i'm writing this post it looks like they have a huge storm with loads of wind over there.

Adak is located on a small Adak island in Alaska, which by the way used to be a military base during cold war. Now it seems to be an abandoned ghost town. Read more here:
<a href="https://www.thealaskalife.com/blog/adak-island-abandoned-alaska">https://www.thealaskalife.com/blog/adak-island-abandoned-alaska</a>

<h4>Weather song generator</h4>
Generates midi files (songs) based on weather forecast for given city.

It's using <a href="https://openweathermap.org/forecast5">OpenWeatherMap API</a> to pull weather forecast for next 5 days (3 hour) for specified location. This gives 5 days x 8 points = 40 weather data points, on which the midi file generation is based.

To generate a song, download <a href="https://github.com/madhr/py-weather-music">py-weather-music</a> to your computer and run python script with specified city name and country code being ISO 3166-1 compliant, example:

```python main.py kielce pl```

So how was it made?

<h4>Process</h4>

The weather API gives us various data around weather. I decided to pick five measures that felt easiest to work with. I came up with this setup:

<ul>
	<li>
		<b>Temperature</b> - the leading melody, represented as up & down arpeggiator, using given music scale, notes pitch range based on amplitude of temperature. I've only noticed now that open weather api only gives very detailed temperature info for 1 day, the remaining 4 days are very general, hence the arp pattern sounds interesting at the beggining and tends to stay on the same range of notes later on.
	</li>
	<li>
		<b>Rain</b> - the more rainfall, the wider pitch range for it. I've randomized note length to give it a feeling of lots of small raindrops.
	</li>
	<li>
		<b>Clouds</b> - clouds can feel massive and heavy, so they're represented as chords. The more clouds (in %), the longer chord length.
	</li>
	<li>
		<b>Humidity</b> - high humidity feels intense, so I decided that heavy bass notes feel like it. The bigger humidity (in %) - the bigger velocity of bass notes. This is a quarter note being perfect unison, so that we know where the rhytm section is. This also means that we won't be having any rhytm section if the air is dry. Sorry not sorry.
	</li>
	<li>
		<b>Wind</b> - a perfect unison note, higher wind speed means longer note playing. Picked a noisy and overdriven sound for this.
	</li>
</ul>

It's worth mentioning that song tempo also reacts to the weather. It's slower when it's colder, and faster when it's warmer - just like atoms.

All params are set to fit the given music scale, so that your ears don't start bleeding. :)
<br>

<h4>Visualizing midi output</h4>

Here's another example for a place circulating between -40&deg;C and -35&deg;C. Town called Deadhorse, located in Alaska (again). I don't know if I'm a great fan of Alaska, but I must admit they have intersting weather conditions to work with. 

<iframe width="100%" height="150" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/761138419&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe>
<br><br>
This is how it looks like when we import midi file into midi editing app: (Ableton Live in this case)

<img src="{{site.baseurl}}/assets/img/deadhorse.jpg" style="width:100%"/>

There's no rain at all and no significant temperature amplitude shifts. Some clouds, some humidity and a bit windy.

Let's look at another location:

<iframe width="100%" height="150" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/761138401&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe>
<br><br>
That's in Norway, city called Stavanger. Look at the Ableton view:

<img src="{{site.baseurl}}/assets/img/stavanger.jpg" style="width:100%"/>

Temperature is shifting a lot, wind blows, and it starts raining around 0:20 of the track. Clouds are present too.

<h4>More examples</h4>


Kano in Nigeria
<iframe width="100%" height="150" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/761138410&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe>
<br><br>
Riyadh in Saudi Arabia
<iframe width="100%" height="150" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/761138407&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe>
<br><br>
See full code at <a href="https://github.com/madhr/py-weather-music">github</a>