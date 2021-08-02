---
menu: Visual Accessibility
tab: false
parent: Engineering.md
weight: 5
---
# Visual Accessibility
## Access for visually impaired players
Gaming these days in 2021 becoming much more popular, with steam as a platform peaking around
24 million players, accessibility is becoming more of a need than just a supported feature 
these days. Around 8% of men and about 0.5% of women suffer from colour blindness (colour vision
deficiency) this amounts to nearly 10% of players, around 3.38% of the world population are 
visually impaired with blindness.

## How as engineers can we tackle adding support?
Most engineers reading this know, adding layers of support on top of systems is not easy to do
in a robust efficient way, it takes development time and a lot of solutions can couple them self
in to other systems, which can make things difficult for development.

I am going to showcase some methods and ways we can add accessibility with to a project.

## Accessibility tools and simulating being visual impaired
We can't perfectly simulate what It's like to be visually impaired, but we can put together
tools to simulate things to give us a good idea of what players might be seeing.

### Visual Blur
Having a visual blurr preview is extremely useful, there are many ways to do this but an easy 
way is to leverage a UMG blur widget over the screen.
![Blurr](Media/Blurr.gif?raw=true "Blurr")
This allows us to simulate how someone with blurry vision may see the game, this helps to reflect
a lot on things like the geometrical shape of levels and how colour themes could be blending.

### Colour Deficiency
Using a colour LUT we can apply Post processing over our scene to view things from colour deficiency 
perspective.
![Blurr](Media/ColorLUT.png?raw=true "Blurr")
``These can be found for free in the epic 4.26 accessibility demo``

Setting up a tool to cycle through the post-processing colour LUT is an effective way to easily view
the project and once again reflect on colour themes and how things could possibly be blending.
![Blurr](Media/ColorBlind.gif?raw=true "Blurr")

## Solutions to add multiple types of support that cooperative
### Entity Highlighting
Entity highlighting is a great way to add definition and clarity to many types of visually impaired 
people.
![Blurr](Media/Hilight.gif?raw=true "Blurr")

Using a material parameter collection, we can control a set of vector4 variables that can be used to 
apply base colour changes to the base colour and emission, these can help to highlight and focus important
objects for people who are struggling to define entities in game.

![Blurr](Media/MaterialParam.png?raw=true "Blurr")

This sort of control could even be interlinked in to a mastery material using a switch system to easily
define what type of entities get what offset value.

![Blurr](Media/MaterialInstance.png?raw=true "Blurr")

``note this is just an example and not an exceptionally efficient way to do this, its possible doing it
this way and applying changes to the master material(s) could cause cascading shader recompiling``

### High contrast Entities
Applying a high contrast post processing can be a great way to really help define important entities
in the scene, as using just colour entity highlighting might just not suite everyone's needs.

![Blurr](Media/HighContrast.gif?raw=true "Blurr")

We can use a custom stencil value from custom depth pass to drive a post-processing material to saturate
out of the colours of entities not in our specified stencil range, this can be achieved pretty easily.

![Blurr](Media/DepthPP.png?raw=true "Blurr")
``note this method has limitations and does not cooperate with transparent materials``

### Pathing - Highlighting entrances
When it comes to pathing we want to be able to effectively and quickly mark pathing visual support down
without ending up with the support becoming coupled with systems like doors, one way to tackle this is
by having a dynamic actor, you can place in your doorway that will use some material magic to apply support
for door highlighting and high contrast support.

![Blurr](Media/PathingDoor.gif?raw=true "Blurr")

I create an actor that can be easily dragged and dropped in, and all the designer has to do is is scale them
approximately around the entrance. These also carry functions that can be leveraged to toggle them off and on
so things like locking doors etc can easily hook in to neighbouring path entrance actors.

![Blurr](Media/DoorPlace.gif?raw=true "Blurr")

### Colour Deficiency
Luckily when leveraging unreal engine 4 tools, colour deficiency is just a node click away, it requires
very little setup.
![Blurr](Media/ColorBlindNode.png?raw=true "Blurr")

This handy little node sets support for the Deuteranopia, Protanopia and Tritanope, with severity control.
Also options for correct deficiency and show correction.

### Screen reader support
This is another easy one, it requires only to `Accessibility.Enable=1` enabling third-party screen reader
software to be able to read UI elements, this can be customized on accessibility options in the UMG widgets.
![Blurr](Media/ScreenReader.png?raw=true "Blurr")

```note: as of 4.26 this is classed as a experimental feature and Epic do not recommend shipping projects with it```

## End notes

There are lots of other things that can be done to help give support, individual or group UI scaling, audio cue's on
UI navigation, disabling disruptive effects like shake and flash etc
Having a good core system that will play cooperatively with visibility support is very important as multi layers
of visual support will be needed by some individuals.

![Blurr](Media/Multi.gif?raw=true "Blurr")