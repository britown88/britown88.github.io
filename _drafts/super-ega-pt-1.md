---
layout: post
title:  "Super EGA"
---
Ever since I started this space, I've been meaning to sit down and talk a bit about the nature of my current project. Being as my bursts of hard work on the thing are often brief and sporadic, it's difficult to nail down a few hours to just document the endeavor.  Now that it's been 14 long months and four whole other entries to the blog, I'm forcing myself to channel this most recent momentum into a little explanation.

## What's all this then
This all started in fall of 2014 when I was toying with the idea of making a game in C as purely a journeyman endeavor.  I originally learned to program with C but it had been years since I coded without the comfort of C++ classes and the STL.  The game itself was going to be a text adventure with a robust modern parser but over time it shifted into being a graphical point & click adventure game.

I love old games, plain and simple.  I have a bit of a romantic view of the games industry of a quarter-century ago.  I am ceaselessly fascinated by the innovations and workarounds of developers of the late 80's and early 90's who made incredible things happen on what we now consider primitive hardware.  With the rise of the modern independant game we all saw a pretty dramatic resurgence of the pixel sprite.  "Retro" quickly became an art style as some titles shot for pure nostalgia while others experimented with elaborately complex pixel art that wouldn't have been feasible on old hardware.

![](/images/posts/super-ega-pt-1/shovelfire.gif "Shovel Knight (2014)")

Despite wildly differing opinions on whether or not this trend is _good_, few can argue that the exploration of the concepts has lead to some truly wonderful titles.  One particularly special entry was Shovel Knight, an extremely popular NES-inspired platformer from 2014.  Shovel Knight captured the look, feel, and magic of a mid-to-late era NES title while still employing modern design concepts to keep it fresh and near-universally approachable.

The thing I enjoyed the most about Shovel Knight was a Gamasutra post from the developer entitled [Breaking the NES for Shovel Knight](http://www.gamasutra.com/blogs/DavidDAngelo/20140625/219383/Breaking_the_NES_for_Shovel_Knight.php).  I highly recommend reading but the gist of it is that the stated goal of the game was to faithfully reproduce a "rose-tinted memory" of the NES.  Yacht Club Games put forward a remarkable amount of research effort into really understanding what made the NES tick.  By understanding the unbreachable, hardware-defined laws for what the system was capable of, they could then decide on which ones to tweak or altogether _break_.

The work behind Shovel Knight showed a care and reverence toward old tech that I enjoyed and appreciated deeply.  It got me thinking about my own "old game," the aforementioned C-based adventure game project.  I will always hold a special place in my heart for the old Sierra and (later) Lucasart adventure games, particularly King's Quest 1-3 and Hero's Quest (later renamed to Quest for Glory).  Before too long I started toying with the idea of my game looking and feeling like it was released for DOS in 1988.  It was around this time that I fell into a deep and dark hole of researching old computer [graphics hardware](https://en.wikipedia.org/wiki/Video_card#History) and [display standards](https://en.wikipedia.org/wiki/Computer_display_standard).

## CGA, EGA, VGA: A Brief History
### The rise of the IBM PC

<img src="/images/posts/super-ega-pt-1/us__en_us__ibm100__the_pc__icon__540x324.png" style="width:400px;display:inline;float:right">

12th of August, 1981.  The Original IBM PC, Model 5150, entered the infant "microcomputing" market[^1].  It was considered a play by IBM to attempt to compete in a space largely dominated by Apple, who enjoyed legendary success with the Apple II in 1977.  The 5150 sported 64 kB of memory, a 4.77 MHz Intel 8088 Processor, a 5.25" floppy drive, an on-board PC Speaker, and retailed for 2-3 thousand dollars. Perhaps most importantly, the cheapest and most popular operating system available for the 5150 was PC DOS created by the 5-year-old Microsoft on commission from IBM.

The details of IBM's commission allowed Microsoft to go ahead and sell their own version of PC DOS, called MS-DOS, for use on non-IBM computers. IBM then encouraged software developers to write programs for DOS, rather than specifically for the IBM PC hardware, with the goal being that that software could run on any machine running DOS.  However, code accessing the hardware directly was universally faster than going through a standardized software layer.  Given the overall success of IBM PC sales, developers were justified in taking advantage of this increase by making their software, especially games where speed is perhaps most crucial, target IBM PC hardware directly.

What followed was the age of the IBM PC-Compatible Computer.  Microcomputer manufacturers, in order to compete, created machines that both ran MS-DOS and mimicked the hardware needs of its most popular software.  By 1983, the market was beginning to fill to capacity with "PC Clones" from companies like Compaq, Tandy/Radio Shack, Hewlett-Packard, and Texas Instruments. When supply and the pricetag of the 5150 and its successors failed to meet demand, many users opted for the cheaper alternatives.
![](/images/posts/super-ega-pt-1/maccharlie_large.jpg)

In a flurry of competitive one-ups-manship, the home computing market of the 80's became one in which, for the first time, software developers could begin to rely on a fairly encompassing install-base for the software they wrote. Most home computer owners could buy and play your PC DOS game.

### Graphics Standards and the Video Games who loved them
#### CGA
The 5150 came equipped with IBM's first ever graphics card, the _IBM Color/Graphics Monitor Adapter_.  The card introduced a graphics standard which would be come to known as simply the Color Graphics Adapter, or CGA.  This card boasted a stunning 16 kB of _dedicated_ video memory which allowed it to render either a resolution of 320x200 with 4 colors from a 16-color palette, or an astounding 640x200 resolution with 2 colors.

Color in CGA is 4-bit. I'm going to try not to go too deep into how binary data is stored or read here so hopefully if you're not up on your bitwise, it'll still make sense.  4-bit color in this scenario means that every color the card can display is composed of 4 boolean, yes/no values.  These are red, green, blue, and intensity.

{% include cga-table.html %}

As you can see from my highly flamboyent chart, by using every possible combination of all 4 bits, you come out to 16 possible colors to display.

"But Brandon", you might say, you estute reader you, "You said that the CGA card had 16 kilobytes of video memory and if you have 4 bits for each pixel then that would need 320 x 200 x 4 = 256,000 bits! That's 32 kilobytes, not 16!"

Right you are. But if you paid close attention, you may have also noticed that I didn't say it rendered with 4-bit colors, I said it rendered _with 4 colors_.  Every pixel in video memory was only given _2 bits_ whose possible values can only be 00, 01, 10, or 11 or, in decimal, 0, 1, 2, or 3.  Since every pixel used 2 bits, not 4, you only need 320 x 200 x 2 bits, which is 16 kilobytes on the dime.

How then, is CGA considered 4-bit color?  For that we need to talk about palettes because they'll come into play for EGA and VGA as well.

#### Palettes
CGA's 4-bit color allowed a screen to show any of 16 colors on a pixel on the screen.  But there wasn't enough memory for all 64,000 pixels to each have its own value from 0 to 15.  Instead, each pixel can only have a value from 0 to 3. To get around this, the user chooses 4 of the 16 available colors that will be displayed at a given time.

For example, if I chose dark blue (2), dark green (4), dark red (8), and white (15) as my 4 colors, I can now render some 2-bit pixels.  Each pixel has a value 0-3 so we can use that value as indices into my list of 4 chosen colors.  Pixels with a value of 0 will be rendered with dark blue, 1's as dark green, and so on. In display standards, this set of chosen colors to use is called a Palette.

The important thing to remember about a palette is that I can change the colors in that palette very quickly just by modifying my 4 4-bit palette values.  The next time the screen refreshes, all the values of the pixels will be unchanged but will now display different colors because their palette indices are pointing at different colors in a different palette!  If you've ever heard the term 'palette-swap' to refer to similar images being reused in a game with altered colors, this is the same concept.
![](/images/posts/super-ega-pt-1/pal-1.png) ![](/images/posts/super-ega-pt-1/pal-2.png)

[^1]: Companies began to settle on the term Microcomputer to distinguish the new machines from the popular "Home Computers" of Atari and Commodore from the 70's, which had a recognition of being budget and not very powerful.
