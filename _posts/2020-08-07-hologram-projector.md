---
layout: post
title: "Intro to Integral Imaging for Hologram Game"
date: 2020-08-07 01:00:00
image: ''
description: "This is not a Computer Science class, I swear."
tags:
- 
categories:
- Game Development
twitter_text: "Intro to Integral Imaging for Hologram Game"
---

<img src="https://media.giphy.com/media/3oEhmPyqvHwsGqY9Xi/source.gif"/>

Just a bit of disclaimer, I'm not an expert in this field. This is simply me sharing what I know about the matter. I just found out about this around February this year, because one of my professor sort of 'challenged' my friends and I to create a hologram game. We didn't really know how to simply make a hologram game, let alone projecting a simple hologram. So we did a little digging into the good ol' Google Search and found out a method called "Integral Photography" or Integral Imaging, as [Wikipedia](https://en.wikipedia.org/wiki/Integral_imaging) called it.

### The Defining

The definition Wikipedia gives, still makes no sense for me. Then again, I barely understand half of the jargons written in there, but here's what I can understand so far: Integral imaging is an imaging technique that enables people to see 2D stuff on screen as 3D when projected and viewed at certain angles. The original Integral Imaging technique uses what's called the "Fly's eyes lens", don't really know where you could get that. This type of lens, when lined up can translates a 3D object into 2D and re-project that 3D object.

But, just re-projecting a 3D object into a screen doesn't make it a hologram. We need what is called Autostereoscopic display (I know, that's a mouthful). This type of display allows users to see dimensions and depth of objects inside their screen at angles. Here's a visualization to get you started:

![](/assets/img/integral-imaging/display.png)

Which reminds me of that old 2002 iMac G4 display which follows your sight or whatever. Yeah, kind of like that but instead of the screen, it's the images inside the screen that 'follows' you.

So how do you build a display like this? Well, financially speaking it's going to be expensive as heck. Reason? A circle has 360 kind of angles, and that's just on one 2D plane. Now imagine turning that into a ball to make the perfect, Iron Man movie-grade kind of hologram? I don't know, abundance, you get the idea. So instead of wasting away thinking about how to create a display like that and revolutionize an entire tech industry, we can choose a simpler option: **limit the angles**.

Sounds like a bailout? Yeah, I guess so. But in this project we mainly want to focus on trying to finish the game side, not trying to create a breakthrough in the fields of computer vision or something, I don't know. So restricting the amount of angles sounds like a good deal for me.

How many angles then? Good question. The answer is: it depends. Depends on what kind of 3D shaped object you can easily recreate. So in my case, I went back and did some Googling on the matter. After two days worth of 24/7 running Chrome with 50-something active tabs on my laptop, I found this [blessed piece of conference paper that just shatters my heart and soul, making my eyes watery with tears of happiness](http://avestia.com/EECSS2016_Proceedings/files/paper/MHCI/109.pdf). Spoiler: it's a pyramid (\*X-Files theme song playing in the background\*). I read the title, bookmarked it, and slammed my body onto my bed, because I had been surviving on 3 hours of sleep every night ever since this project started.

So, creating a square-based pyramid shaped object is not entirely impossible. It is manageable and not an out-of-this-world concept to begin with. It's solid. But with what material?

Coming from what we already know about Integral Imaging, we simply don't need to translate a 3D object into a 2D plane. Any 3D game engine already does that for you. So what we need is a **projector** that will 're-project' those 3D game scenes you see in your screen. Okay great, a projector. Let's read through more papers before deciding which materials. 

**\*10 hours later...\***

Apparently the best material is something that's glossy and obviously clear-colored (is 'clear' a color?). Plexiglass/acrylic is among the most used material in this case, apart from actual glass.

### The Creation and The Result

The dimensions of the acrylic will solely depend on the screen you're going to use as the projection source and the size of the 'images' you want to project. Just make sure, since it's a square-based pyramid, you'll going to need 4 2D planes as well (better in four different angles as well) so it can be projected and created this so-called one 3D object which can be seen on all four sides of the pyramid! Something like this:

![](/assets/img/integral-imaging/projector.png)

Credit goes to my friend for creating this model to be used in a thesis we're writing together. Using a game engine can help a lot on setting up the positioning of the image. In this project, we use Unity. Simply create multiple cameras, and Transform them. Set the X, Y, width and height; the usual drill. 

Each scene shown by each camera will show your game scene so you can project your 3D game scene using the pyramid! If you want to get a transparent background, use black color in your game scene, since black doesn't project any light, it is considered 'transparent'. That's why apart from the scenes, I set the world background color to black.

![](/assets/img/integral-imaging/unity-1.jpeg)

(Excuse my poor screen quality) Make sure they fit really good with each other, so there won't be any gap or distortion on the final result of the projection. So you'll get something like this: *(look mom, I made something cool)*

![](/assets/img/integral-imaging/result.JPEG)

It still doesn't look like much. This is still a prototype projector in an ongoing project. I'm working on building a better one, bigger in size and taller so it's easier to look at. But, the possibilities of this holographic pyramid projector is abundance, just google them. I've seen local museums integrating this kind of things into their gallery for a more immersive tour experience (?)

And that wraps up this post. Thank you for sticking out until the end. This is my first blog post of sharing such technical stuff like this. I hope this can inspire(?) or get you started on something which might need this kind of information. See you in another post!

<img src="https://media.giphy.com/media/jSuu6dQWhSEQE/source.gif" />

#### Reference and Creds:
- [Integral Imaging by Wikipedia](https://en.wikipedia.org/wiki/Integral_imaging)
- [Autostereoscopic 3D displays by Neil A. Dodgson](https://www.semanticscholar.org/paper/Autostereoscopic-3D-displays-Dodgson/7dc153e1a3ca16a52a2ae5ba115515032d004fea)
- [Holographic Pyramid Using Integral Photography](http://avestia.com/EECSS2016_Proceedings/files/paper/MHCI/109.pdf)