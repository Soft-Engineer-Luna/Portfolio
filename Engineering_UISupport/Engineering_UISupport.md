---
menu: UI Support
tab: false
parent: Engineering.md
weight: 4
---

## Wide ratio Support
In 2022 the popularity of different aspect ratio monitors are rising, one of the most popular 
type of monitors for gaming are Utrawide and Super Ultra-wide spanning an aspect ratio of 21:9 & 32:9.

Imagine this, you buy a new game for 50$, you boot it up, and you set your resolution support, next thing
you know your UI is so far apart you need to rotate your head a good amount to be able to see things 
properly, what do you think is going to happen next? Are they going to change their resolution of there
$1000 monitor or are they going to just quit the game? Most of the time this can go nasty, no one wants
to play not in their native aspect ratio. 

This is such an easy thing that can be over looked and alienate a ever-growing section of hardware in the 
pc industry.

## What is true wide support

Many people will say that adding wide support is just making sure anchors for UI are in place properly, but 
i disagree, just having anchors in the correct place is not enough. As nice as wide monitors are a lot of the
time the extra vision you gain is not actual focus vision you can pay attention too.

![Tiling Pattern](Media/UWExample.png?raw=true "Tiling pattern")

This is not good enough, as we can see in this example our weapon bar, and mini map for this super ultra-wide
ratio 32:9 are way too far apart, this is not really viable for primary visual feedback for the player.

### The Problem
Lets take a little look at a curved ultra-wide monitor, and take notice to what is in focus vision.

![Tiling Pattern](Media/SUW_View.png?raw=true "Tiling pattern")

As we can see from the view cone, i have highlighted the average viewing area that is in focus, and the outer area
are still in vision, but its hard to read important data from the sides.

### Solution
A good solution for this is enabling padding to shift the UI tighter towards the focus area, this can be a very
effective way to handle this, however, it requires that are UI is set up correctly with good resizing support,
a good example of this is using as little static UI transform data as possible and enabling slot system to correctly 
size things, without this shifting the UI could turn out with overlapping UI or undefined behaviour.

``small example of scaling UI``
![Tiling Pattern](Media/ScaleUIExample.gif?raw=true "Tiling pattern")

AS long as the UI is set up to scale well, then we can create a UI wrapper to handle padding the sides.
With this we are able to give the user the option to compress the UI to have better vision focus for 
them without having to make the whole UI a modular system, which can be a monolithic task depending on
on the game.

![Tiling Pattern](Media/UWFix.gif?raw=true "Tiling pattern")