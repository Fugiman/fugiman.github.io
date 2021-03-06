---
layout: post
title: "On click-fueled javascript games"
date: 2013-08-17 04:35
comments: true
categories: games
---
*alternatively: ClickQuest and Cookie Clicker, A tale of two games that start with a single click*

Before I start, I recommend you at least glance at the games for a minute or two.  
<http://www.clickquest.net/>  
<http://orteil.dashnet.org/experiments/cookie/>

I made ClickQuest almost exactly 3 years ago, far past the time required for me to regret every line of code I wrote.
However, I did learn quite a few things in its bug-riddled construction, and many things in the years of reflection afterwards.
ClickQuest was originally marketed as "distilling an MMORPG into its most core mechanic - click and numbers go up".
Indeed, Cookie Clicker takes this a step further: Click to increase a number that you can decrease to increase faster.
It is the most instant of gratification, a split second is all it takes.

However, while this makes the games extremely good for single-player, it makes it incredibly hard for multiplayer.
If you play by yourself, there is no reason to cheat, as all you do is lower your own enjoyment.
When multiplayer (more specifically a leaderboard) is introduced, then players are motivated to cheat.
In MMORPGs the layers of abstraction provide a mechanism to defeat cheating.
But in clicking games, we've peeled away those layers, and so our only action in the game is clicking.
This is the biggest strength and greatest weakness in clicking games.

When I made ClickQuest, it was designed with a leaderboard and chat in mind.
I knew that cheating would be a big concern, so I tried to proactively thwart it.
Here's what I tried in order of when I tried it.

### Approach 1: Send every click to the server

This worked great in testing.
If I saw every click then nobody could fake it, right?
In reality, this just changed cheating from faking clicks to spamming my server with HTTP requests.
And when your game generates a request for every click, you've just invented a way to DDOS your own server.
Oops.

### Approach 2: Rate limit clicks client-side

I changed the client to only accept a click when 60ms had passed since the last click.
I found this was a reasonable upper bound for how fast a human can click a mouse, even with a mouse in each hand alternating clicks.
(Yes, I had some hardcore players.)
But this just slows down the cheater, assuming they don't go past it straight to the server.

### Approach 3: Obfusicate the request payload

Aha, if players can bypass the client to cheat, I need to force them to use the client.
Simple! Make the packet sent from the client to the server super special secret so that they can't duplicate it!
And of course, obfusicate my javascript so they can't find out how I do it!
A great idea in theory, but it is [pretty trivial to unpack javascript](http://jsbeautifier.org/).
In reality, this did nothing.

### Approach 4: Track as many metrics as I can and figure out where they should be server-side

Finally, a proper solution.
I kept track of how many clicks per second a user made whenever they made a request to the server.
I'd then compare that to how many clicks per second they'd made in the past 100 requests.
If your clicking was consistently close to the 60ms per click mark, I'd give them a strike.
Likewise if they clicked faster than 60ms mark, or if they were clicking at the same rate constantly.
This actually led to catching bots, but unfortunately it had too many false positives and had to be turned off.

### Approach 5: Give up and let it happen

By this point I'd spent 6 months and had no idea what else I could do.
It is extremely difficult to regulate pure clicks.
If I were to attempt it today, I'd use the efficiency of websockets to combine approach 1 and 4.
I'd log the location and time of every click for every user, and use that to try and weed out (poorly designed) bots.
But even that would likely fail.
The developer simply lacks enough information to properly prevent cheating.

### But Cookie Clicker is different than ClickQuest!

Orteil has added back a layer of abstraction in the form of buyable cookie producers.
Now, past a certain point, it's the buildings producing 99% of the cookies - clicking is irrelevant.
Using my numbers a player can legitimately acquire 20 cookies per second from clicking.
If you open a websocket connection for each player, you can track whether they have the page open or not.
Then, send a packet every 30 seconds, or on building purchase (whichever happens first).
Each packet would have the number of cookies and each type of building.
You can verify the buildings with stored data, and the cookies with simple arithmetic.
Boom, cheating solved!

But not quite.
You've just slowed botting down to human levels.
Like an MMORPG, it's possible to bot as long as you play by the rules.
From here, you'd have to use browser fingerprinting, or other esoteric metrics to weed out bots.
Just remember, the stricter your anti-cheat protection is, the likely you are to anger your legitimate players!

### A few parting thoughts

I love javascript clicking games.
They seem extremly simple, but can hide great ingenuity.
(Also, they ease my pain of not knowing flash).
But I feel you need to know what you're getting into when making one.
Unlike flash, or a desktop app, it's easy for the most basic of programmer to read your entire source code.
A little mucking around in dev tools and they can pry your game apart.
I find this a great thing, as it allows others to learn from what they find, but it can lead to the problems I mentioned above.
Again, just know the limitations of your platform.
