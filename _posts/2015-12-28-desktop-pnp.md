---
layout: post
title: The quest for desktop pcb assembly
---

![](/images/pnp.jpg)

PCB Prototyping is one of the most broken things in my daily job as an electronics entrepreneur.
I built *alot* of prototypes for my various hardware startups such as [Hy5](hy5.berlin) and [feral](http://www.horse-analytics.com/) etc.
While PCB itself is a figured out thing, thanks to the likes of [oshpark](https://oshpark.com/),
PCBA is still a matter of which poison you prefer.
Hand building is annoying, slow and absurdly inefficient.
I can't get past 50% yield time hand picking 0402 components. Some people are really good at this,
but i'm more of a hacker than a real electronics engineer anyway.

On the other side of the spectrum is going to a professional factory,
such as my amazing friends at [sizzl](http://sizzl.berlin/) or someone cheaper
like maybe seedstudio. they're established industries trying to get into the startup world now by
being more accessible.  I applaud that, but i'm still jealous of the industrial designers,
who can just throw stuff into their inhouse 3d printer.

I want a factory on my desk. That said, everyone wants that, so obviously someone had to do it eventually.
And someone did: [http://delta.firepick.org/](http://delta.firepick.org/). It looks good, but i couldn't figure
out how they want to achieve the kind of precicion i need with a delta design. They are however based on an
amazing open source project that definitely looks very solid, called [openpnp](http://openpnp.org/)
It even has a reference machine design based on openbuilds.

While that's certainly fun and entertaining, i have a different goal. I want a *commercially viable machine with
0.1mm precision well below 1K*. The kind of price range at which everyone can just have one at home, office or university. 
Since i'm a hacker, i went and build one. Here it is placing 0603:

<iframe width="720" height="405" src="https://www.youtube.com/embed/FtclhOmzjW0" frameborder="0" allowfullscreen></iframe>


The prototype cost me around $500. That's half the reference design. A large cost factor is of course ordering stuff at low quantities and shipping from china.
The hack also reuses some very poorly designed electronics, because all the stores where closed on christmas, which makes it so ridicously slow.

next steps:

 - solder paste dispensing
 - design a controller board
   - alot less wires
   - runs openpnp directly
   - with wifi
 - getting cost down
 - making it pretty
 - redesign the entire head really

The head is amazing for its cost and availability (about $15 worth of stuff from local stores), but nowhere the quality i want it.
The glorious Jason von Nieda has plans to sell a whole head and i want to focus on electronics, so i see some room for a good deal here ;)

That said, this is already usable and will replace hand picking in my lab. Not bad for a 7 days christmas hack.

If you want to rebuild my design, i am happy to write up detailed instructions. Send me a message.
If you're looking to buy a complete machine, send me a message too. Won't be available anytime soon,
but of course i'll prioritize it, if enough people want one.

