---
title: "A Game So Nice I Made It Thrice"
date: 2026-01-20
---

![Cube Boi](/notablog/docs/assets/2026_01_20_cubeboi.jpg "Cube Boi Model")

In my hobbyist game developer persuits I recently dove into the wild world that is trying to learn 3D modeling in Blender.  Luckily there are countless resources out there to help - one of my favorites being the YouTuber Grant Abbitt.  I was going through a course of his and did a speed run on his character that I have since named Cube Boi and thought to myself, "I wonder if I could figure out how to animate this simple model and then control it in a game engine?"

[Grant Abbitt's YouTube Channel](https://www.youtube.com/@grabbitt)

# TL;DR

Can you tell which is which?

<video controls playsinline preload="metadata" style="max-width:100%; height:auto;">
  <source src="/notablog/docs/2026_01_20_cubeboi.mp4" type="video/mp4">
  Sorry, your browser doesn’t support embedded videos.
</video>

# Animating

Like many endeavors past, I had no earthly idea what I was doing when I decided to animate Cube Boi.  Countless tutorials later I added a simple Armature to the model:

![Cube Boi](/notablog/docs/assets/2026_01_20_blender_armature.PNG "Armature")

I decided to keep things simple and start with a walk animation which basically involves setting a few key poses in an Action sheet.  This might be the most simplified walk cycle of all time but hey, it works:

![Cube Boi](/notablog/docs/assets/2026_01_20_walk.gif "Walk Cycle")

# Engine Selection

At this point it was time to fire up a game engine - but which one?  On one hand, I spent a decent amount of time in Unity developing my previous game.  But on the other hand, a simple game is a good excuse to learn something new.  So I started leaning toward Unreal.  But I enjoyed learning a bit of Godot a couple years back so maybe it was worth revisiting...enter analysis paralysis.

Rather than overanalyze the process any further I decided to do what anyone with entirely too much time on their hands would do: make the same game in all three!

# The Game Mechanics

My justification for this seemingly ridiculous process was simple: after implementing a simple game in the big three, I could make an educated decision on which engine I find myself most capable in.  This could be the engine I move forward with for larger projects in the future.  As such, I decided on the following universal criteria for my game:

-Create a new project in engine of choice
-Import CubeBoi FBX/glTF file with animation data
-Enable movement of the Cube Boi character with animations
-Create terrain based on Gaea output
-Add grass texture to terrain
-Add two tree types: one with collisions and one without
-Import coin mesh and add two-shade gold material using mask
-Add logic to collide with/collect coins
-Add logic to "win game" when all coins collected
-Add "start" button on game start that drops Cube Boi from the sky
-"Drop" the coins from the sky to the ground after start button pressed
-Rotate the coins indefinitely
-Start a timer when Cube Boi collides with terrain for the first time
-Use timer as a stopwatch until all coins collected
-Increase timer size as time progresses; change color to red
-Send the exact color to the win screen to display player time
-Present win screen with player time to win
-Slow down character movement when win screen is shown; disable jump
-Allow the option to play again

# Unity

I started out in Unity because I felt the most confident in my ability to quickly implement game mechanics.  I understand the game object -> script that enables said object to do stuff paradigm quite well and tend to enjoy it.  There were a few gotchas here and there, mostly with the UI.  But overall the process was smooth and I finished the entire gameplay implementation pretty quickly.

![Unity Project](/notablog/docs/assets/2026_01_20_editor_unity.PNG "Unity Editor")

Writing/generating code for Unity is a breeze.  It uses C# which is well documented and LLMs are very proficient at it.  I personally like to keep pretty precise control on the scripts I create and attach to game objects, so rather than go full-on vibe coding I create my scripts with purpose and ensure functionality makes sense.  Here is a snippet of some C# code I use for my coin collection logic; this triggers an animation when the player collides with the coin.

![Unity Project](/notablog/docs/assets/2026_01_20_code_unity_coin.PNG "Unity C#")

I think my biggest "gotcha" in the Unity development process was the reflection probes on the coins.  I had never used those before and overall my skills in lighting are pretty lackluster.  I would like to improve on this in all engines since it is incredibly important in games.

# Unreal

Next up, Unreal.  At this point I had never really implemented any game mechanics in Unreal at all.  A few prior learnings include basic YouTube tutorials and a still-work-in-progress rendition of a Halo map but that is a story for another day.

![Unreal Project](/notablog/docs/assets/2026_01_20_editor_unreal.PNG "Unreal Editor")

In hindsight maybe this was a mistake for a fair engine comparison, but I chose a Blueprint-based project at the start.  Because of this, I wrote/generated literally zero lines of code for the Unreal version.  This was pretty insane to me because I figured there would be at least some requirement somewhere to dive into C++.  But no, Blueprints handled all of it.  That is both a pro and a con for a variety of reasons.  It is a pro because, well, beginners don't need to worry about tackling C++.  It is a con because there are a seemingly infinite number of "nodes" within Blueprints to the point that knowing all of them seems like an insurmountable task.  Here is similar coin logic from above, but as a blueprint:

![Unreal Project](/notablog/docs/assets/2026_01_20_code_unreal_coin.PNG "Unreal Blueprint")

Blueprints were definitely the biggest gotcha for me during the Unreal development process.  Developing the game also took me the longest time in Unreal.  I would say that generally, it is because the breadth of functionality that can be implemented by use of Blueprint nodes pretty much anywhere in your game.  Just take a look of how wild this one is, which basically sets up my start screen with one camera and blends it into another camera after the start button gets pressed (among other things):

![Unreal Project](/notablog/docs/assets/2026_01_20_blueprint_unreal_camera_spaghetti.PNG "Unreal Blueprint 2")

It is worth noting that LLMs can help craft Blueprints but at the end of the day they are wrong very frequently due to deprecation, renaming, etc. in incremental release of Unreal.  That said It is truly incredible how powerful this visual programming paradigm is.  I hope to become more proficient at it as time progresses.

# Godot

I had not opened Godot in approximately two years prior to this project.  I basically forgot its paradigm, but quickly remembered that everything is a node and there is a lot of nesting.  It is similar to Unity in some ways that I like, specifically the way you attach scripts to nodes to implement gameplay mechanics.

![Godot Project](/notablog/docs/assets/2026_01_20_editor_godot.PNG "Godot Editor")

I think my favorite thing about Godot is how lightweight the editor is.  It loads up quickly and suits simple games because of it.  I also tend to enjoy the python-like syntax of gdscript.  It is very easy to understand and debug.  I also must say that loading up the Cube Boi model in Godot was the least painful of the bunch.  It just kind of...worked.  That said, I did have to create a Cube Boi scene based on the model in order to modify individual components.  Doing so from the .glb file directly is a no-no and the editor will make that pretty clear.

![Godot Project](/notablog/docs/assets/2026_01_20_cubeboi_godot_scene.PNG "Godot Cube Boi Scene")

I think my biggest gotcha in Godot was really the lack of maturity in building terrain.  The terrain sculpting, painting, etc. feels much more mature in both Unity and Unreal.  Importing a heightmap from something like Gaea is trivial in those engines.  You can do it in Godot but I ended up bailing because it was taking entirely too long and seemed dependent on a plugin called Terrain3D.  It was easier to just re-create my terrain in Blender and import that way.  I also would call out that the Godot asset store is very immature compared to what is available for Unity and Unreal.  I am hoping this improves as time goes on and would love to contribute somehow.

# Final Thoughts

This project was a great learning experience and as you might imagine, all three engines can be a fit for your game depending on what you want to achieve.  My game was simple enough that I could go forward with any of the three engines and end up with a superior product to what I have created thus far given the months of effort required to make a polished experience.

That said, any bias towards Unity notwithstanding, I am just more proficient using that paradigm than I am the others.  So if I were to take the Cube Boi game loop and expand upon it, turning it into a real game (with more than 15 seconds of gameplay), Unity would be my overall choice. 

# Credits

Thanks to the following for providing resources!

Music:

[Playlist 1](https://www.youtube.com/watch?v=TyN6OpPb6ek)
[Playlist 2](https://www.youtube.com/watch?v=9JmhXtq-KOw)
[Track 1](https://www.youtube.com/watch?v=j4mec_OWp-A)

Sound FX: 

[Coin & Jump](https://brackeysgames.itch.io/brackeys-platformer-bundle)
[Win Sound](https://www.youtube.com/watch?v=UdzJBtAdbFI)
[Win Sound 2](https://www.youtube.com/watch?v=3a0pqvHnsGQ)

Blender:

[Cube Character created by Grant Abbitt](https://www.youtube.com/@grabbitt)

Godot:

[Skybox](https://polyhaven.com/a/kloofendal_48d_partly_cloudy_puresky)
[Trees](https://store-beta.godotengine.org/asset/quaternius/stylized-nature-megakit/)

Unity:

[Grass] - https://assetstore.unity.com/packages/2d/textures-materials/nature/handpainted-grass-ground-textures-187634
[Sky] - https://assetstore.unity.com/packages/2d/textures-materials/sky/allsky-free-10-sky-skybox-set-146014
[Trees] - https://assetstore.unity.com/packages/3d/vegetation/trees/free-trees-103208

Unreal:

[Grass](https://assetstore.unity.com/packages/2d/textures-materials/nature/handpainted-grass-ground-textures-187634)
[Palm Tree](https://fab.com/s/8e34851013c4)
[Stylized Trees](https://www.fab.com/listings/b066de06-73b8-4fbe-b30c-468f5bcf7575)
