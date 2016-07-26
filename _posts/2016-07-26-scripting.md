---
layout: post
title: "So you want a scripting language"
---
It's become a popular idea over the years that if you're making a content-heavy game then _you need a scripting language, son_. While everything has its time and place, it's easy to see some major advantages in having asset data drive game logic without needing to rebuild on every change.  Since I rarely learn from my mistakes and am moving forward with the most content-heavy game I've ever attempted, it became time to take a hard look at my options.

### Before anything, what problem are we solving

I am hesitant to _a fault_ of using libraries I didn't write myself. I'll spare you the philosophical details but I tend to enjoy the struggle of working through solutions to problems I run into on my own time.  When it does inevitably come time to save an immense amount of life expectancy by using an external library, I try to make sure that it's actually solving a problem I have or will have.  I'm wary of project creators that list off what they're **using** and staying silent on what they're using it **for**.[^1]

#### Your basic data (data.wad)

When we're talking about scripting we're talking about assets.  In it's most rudimentary form you're using something like json or (euch) _xml_ to straight-up read key-value-pairs into your game to drive what's in the levels; what enemies there are, how much HP a bat has, etc.  When it comes to something like BladeQuest, a mid-90s-jrpg-style joint, you wind up with a _ton_ of data.  Load each song and give them names, every map needs a tile grid, every enemy needs HP, every character needs a statblock, every sprite needs animation parameters. All read in from text when the game runs.

There's a lot to be said of the advantages of a pure-data asset system.  If every file is just filling structs with values in your game it can simplify a ton of architecture headaches.  As soon as you start attaching logic to your data files, you run into problems where things like creation, destruction, and execution are done in ambiguous order and can become a tight weave of knots.

#### Scripting in BladeQuest

For BladeQuest we took a hybrid approach where the scripts were still defining simple values for in-engine objects but these objects were closely tied to specific pieces of logic when ran.  A character in the game, when activated, would provide a list of "Actions" which would run in sequence.  Here's a snippet from one of the scripts from the demo

~~~
addAction (wait 1.0) >
addAction (message "|aramis|: \nHmmm... \nYou don't look like the other guards.") >
addAction (message "You new or something?") >
addAction (message "Recruit: \nShut up! You're not supposed to be here!") >
addAction (message "|aramis|: \nOh I get it. Prison duty was a bit much for a newbie.") >
addAction (message "So they got you up here guarding tapestries.") >
addAction (message "Recruit: \nYeah well if I catch you, that'll show them what I can do!") >
addAction (message "|aramis|: \nListen, kid.  You don't have to prove yourself. \nJust walk away.") >
addAction (message "I promise I won't tell the other guards.") >
addAction (message "Recruit: \nEnough! Bring it on!") >
addAction (startBattle "recruit" false >
~~~

Although it looks like code syntax it's still just data.  When this script is read in it creates a list of actions with the accompanying values and when the script is activated in-game it executes each action in order one at a time.

This approach worked wonders for a majority of the interactions in the game.  A "Script" was just a list of things to do when you touched it and each action handled how it behaved or if it blocked the next action from starting etc. If you've ever used RPG-Maker (Specifically 2003) our system is largely modeled on that overall idea.

Unfortunately we ran into some major hurdles as scripts became increasingly complex to fit the various cinematic needs we wanted.

![](http://i.imgur.com/DXJ7Y3E.gif "From here, killing all the villagers in Albi doesn't look so bad.")

Additionally, every new feature required a new action to be created which lead to a significant amount of code bloat as scripts ever-expanded into being able to do more and more.

#### Coroutines

All our action lists really are in the end are linear [coroutines](https://en.wikipedia.org/wiki/Coroutine).  At its highest level, a coroutine is a function that you can pause and resume which allows you to run several coroutines simultaneously (Run one until it pauses, run the next one until it pauses, etc).  When a coroutine "Pauses" it is called "Yielding."   When a coroutine yields, it immediately exits out of the function and allows the program to continue as it was.  The program can then later return to that coroutine and call resume to tell it to continue where it left off.

Using the BladeQuest example:  Our actions lists are just coroutines that, every frame, allow their current action to execute one frame and then yield so that other action lists can do the same.  If, after updating, the action marks itself as "Done," the action list pops it off and sets the next action as current.

#### So what do we need

What would have been really neat is if our scripts were just coroutine functions in some generic programming language.  This would give us power over run-time resolution of data, querying the game state, looping, conditionals, arithmetic operators, and would all look natural and human-readable to boot.

For instance if I had a regular function like this:

~~~ C
void MyScript(){
   wait(1.0);
   postMessage("|aramis|: \nHmmm... \nYou don't look like the other guards.");
   startBattle("recruit", false);
}
~~~

If that function was capable of yielding during the wait or while the text is scrolling or the battle is happening and still resume correctly with the state intact, that would be perfect!

For [Chronicles](/2015/11/17/heres-what-im-making-with-sega.html), this is the primary power of a fully-fledged scripting language that I want to leverage.  A major strength of the game is that the content of the game should be written, not coded.  It should be easy to write full logic-supported interactions and run them all while the game executable is running.

### Spoilers: It's Lua

[Lua's great!](https://www.lua.org) It's the one you always hear about (usually because of [WoW](http://wowwiki.wikia.com/wiki/Lua)) and somewhat surprisingly, that seems to be because it really is awesome. The whole thing's one fat stack of ANSI-C files with zero dependencies that you can just throw into literally any project as a static library and you're totally good to go. Added bonus: [it completely inflates your contribution numbers on github.](https://github.com/p4r4digm/sega/commit/0b20556b704dc68ef9a32d54df02d260fb9cd185)

From their website one can find numerous well-written pieces of documentation that walk you through the syntax of the language on its own and detailed explanations of how its C API works.  On the scripting side you have a simple and familiar-feeling interpreted language with multiple returns and coroutines built-in. The idea that "Everything is a table" is immediately comprehensible and provides a great rule of thumb for attacking any data organization.

#### The native side

When it comes time to call or expose lua from C, there's a bit steeper of a learning curve but it eventually boils down to a system of stack management.  Interacting with Lua from C means pushing and popping data from a stack and referencing those items via (1-based) indices.  Negative indices count backward from the top (-1 is the top of the stack) and just about every function performs an operation with your lua_State* and your index.

When a C function is called from Lua, the stack already contains the arguments passed in in-order and to return values you push your returns onto the stack and return an integer for how many you're returning.  The most important thing to manage call-to-call is keeping your stack clean and in order.  Traveling down the rabbit hole and doing more complex operations will find you pushing a table item name, then retrieving the data at that name onto the stack, then copying item at index 2 to the top and doing a pcall on 6 arguments that are now in order.  

~~~ C
   //now store the thread in the actor's scripts
   lua_pushvalue(L, 1);//push the actor
   lua_pushliteral(L, "scripts");
   lua_gettable(L, -2);//push the actor's script table

   //get the len, add 1 and push it
   lua_len(L, -1);//push the len
   index = (int)lua_tointeger(L, -1) + 1;
   lua_pop(L, 1);
   lua_pushinteger(L, index);

   lua_pushvalue(L, n + 1);//push the thread
   lua_settable(L, -3);//commit script[#scripts] = thread

   lua_pop(L, 3);//pop the scripts, actor, and thread
~~~

Some functions will pop values automatically, others will push items unexpectedly so your first few lua functions will consist of a lot of Ctrl+F on the documentation for each function call to find out what it does to the stack (the documentation does a great job of being explicit about all stack operations).  Your best friend with large exposed functions is to comment each operation with  the state  of the stack for ease of referencing or maintaining later.

### #include "lua.h"

Although I have nothing but kind things to say about Lua, actually integrating it into my code in a way that I felt made sense and was sustainable and scalable often felt like a fool's errand.  Sure it all works great but when should you create your Lua_State? What files should be read in vs made on the fly?  What order should you require libraries in?  At what point in your game's boot process does it start relying on Lua?  How should your game react when Lua throws an error??  I spent an enormous amount of time rewriting my usage of Lua into a way that I felt would work moving forward.

One of the most important additions to the project was an in-engine console (if you run BT.exe off any build you can get there with the tilde) that simply allows you to execute arbitrary lua commands and print the output

{% youtube hb6pgZX6jG8 %}

In conclusion I'll say that affecting your native-engine game while it's running using typed-in scripts feels like magic.  There's a _ton_ of effort behind making your game script-powered in a way that doesn't crash and burn (especially for an admittedly already over-architected engine like sEGA) but as I progress forward as careful as possible the rewards are stunning.

[^1]: OK I didn't spare you the philosophical details
