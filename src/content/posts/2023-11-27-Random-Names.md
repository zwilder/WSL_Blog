---
title: "Procedural Mushrooms and Random Names"
date: 2023-11-27 11:00:00 -0700
Author: "Zach Wilder"
Categories:
  - Programming
tags:
  - C
  - Goblin Caves
  - Procedural Generation
assets: "2023-11-27"
---

The holidays are always a great time to slack off and work on fun projects -
while eating too much pie and wearing sweatpants for multiple days, of course.
Over the past week or so I've made a few neat things and more importantly
tied the Markov chain word generator into the Goblin Caves project.

The first little thing I made is a procedural mushroom generator.

<!--more-->
{{<fbox>}}
0 Mushrooms.jpg
{{< /fbox >}}

No fancy tricks or code here - the generator (almost) entirely uses code from
other projects. One of the guidelines I try and follow while writing code is to
write "generic" code - making lego pieces that I can build into other models.
I've recently started a project "toolbox" that contains a lot of the data
structures and useful algorithms (lego pieces) I've written, which I can pull
out and slap into any other project. The mushroom generator is a good example of
a project put together with those pieces - I wrote this in a couple hours of
lazy coding, and most of that was tweaking the inputs to get better results. I
have a whole bunch of cool ideas for the Goblin Caves that are going to use a
version of this - eventually. A video demonstration:

{{<fbox>}}
0 mushrooms.mp4 lq-Mushrooms2.jpg
{{</fbox>}}

I then decided to work on using the fancy Markov generator in the Goblin Caves
project. The first place I found a good use for it is to seed the random number
generator. When the game starts it generates a two-word seed, feeds it into a
"hash" function, and uses the result to initialize the [Mersenne-Twister psuedo
random number generator](https://en.wikipedia.org/wiki/Mersenne_Twister)
(`mt19937.h/c` in the toolbox contains the C implementation, written by Takuji
Nishimura and Makoto Matsumoto). The seed word is made of a superlative and a
random name, which makes funny memorable seeds like "PoorestElbeth" and
"CurliestAgdush". Whats cool about this is that if two separate players use the
same seed to start the game, they (should) encounter the exact some "random"
dungeon with the same denizens.  Or, if the player dies and wants to try the
same dungeon again, they would just have to start the game with the same seed.
Super cool. A command line switch was added to set the seed when the game starts
up.

The other, more obvious, use for the word generator is to use it with new player
creation.  I originally had the generator assign a name to a player when they
entered a blank string (nothing) for their name - but quickly decided against
that. A better idea: When a player enters nothing for their name, a list of
names pops up, allowing the player to select one and proceed with pummelling
goblins.  

The challenge then was how to actually present the names - and make sure input
from the player is tied to the random name. I realized this was a good
opportunity to make something that can be used in lots of places in the game - a
dialog box, where a prompt and a list of options can be fed in, and user input
can be returned.

To make this dialog box I first needed a couple of drawing "primitives" - I
already had "string" and "glyph" drawing routines, for drawing single characters
and words on the screen. To draw a box (a rectangle - for some reason I talked
myself into calling it a box because it made sense at the time), I first needed
to be able to draw a line. I wrote two functions (one to draw a horizontal line
and one to draw a vertical line) that place a row of solid colored rectangles on
the terminal screen. A solid box is simply a bunch of horizontal lines right on
top of each other (or vertical lines right next to each other) - easy enough. A
rectangle with a border around it (an "empty" box - see? it makes sense!) is easy
now too - it's just a solid box with another, background colored solid box drawn
in the middle. With those few functions I can now make simple routines to
draw dialog boxes in the screen - yes/no prompts, alerts, etc. 

The menu routine draws a box, fills it with content (options), and then returns a
character keypress corresponding to an option on the screen. Example code of how
this is used:

{{< highlight c >}}

SList *menu = create_slist("A test menu");
slist_push(&menu,"Bottom text string that is linewrapped to fit the menu");
slist_push(&menu,"abcd*");
slist_push(&menu,"First");
slist_push(&menu,"Second");
slist_push(&menu,"Third");
slist_push(&menu,"Fourth");
slist_push(&menu,"Random");

...

// Draw a basic menu, return value stored in result
result = draw_menu_basic(menu);


// Draw a fancier menu
// fgcolor is the color of the text and border, bgcolor is the background color
draw_menu(menu, fgcolor, bgcolor);

// Draw the fanciest menu
draw_cmenu(menu, fgcolor, bgcolor, bordercolor);

{{< / highlight >}}

A menu is just a linked list of strings (`SList` - string list in the toolbox). The first
three strings in the list are:
- The "prompt", or the text at the top of the box. It is displayed as a "bold"
  color. If the prompt is longer than the width of the box, then it's
line wrapped automatically to fit the box. 
- The "instructions", or the text at the bottom of the box. Also line wrapped.
- The characters to use for the options, as a string. This is pretty cool -
  because instead of using "1234" or "abcd", I can use anything I want - "shlq"
for example, if the options are "save" "high scores" "load" "quit".

Then the rest of the strings in the list are presented as options. Something
like the code above would produce this in the game (or anywhere else, the code
is almost generic enough it might even find a home in the toolbox):

{{<fbox>}}
1 Menu.jpg
1 Menu3.jpg
1 Menu2.jpg
{{</fbox>}}

The function returns a character, and leaves the logic checking to whatever
called it. So, in the example above if the user presses 'd', the function would
return 'd', 't' would return 't', etc. Makes it nice and easy to reuse just
about everywhere. Of course, after all this I had to spruce up the player
creation screen. Not sure if I like it, I might go back to the black
screen/prompt. Here is a screenshot of the new player screen, and a video
showing everything in action.

{{<fbox>}}
2 NewPl.jpg
2 NewPlayer.mp4
{{</fbox>}}


The latest code for Goblin Caves is on [Github](https://github.com/zwilder/goblincaves/), and the
menu code is in [`src/draw_menu.c`](https://github.com/zwilder/GoblinCaves/blob/master/src/draw_menu.c) (clever, right?)
