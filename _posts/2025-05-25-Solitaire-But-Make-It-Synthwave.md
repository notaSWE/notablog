---
title: "Solitaire But Make It Synthwave"
date: 2025-05-25
---

![Synthwave Solitaire](/notablog/docs/assets/2025_05_25_screen.JPG "Logo")

Late last year I opened up Solitaire in Windows 10 for nostalgia purposes and was horrified to find that it is ad-supported.  Yes, that Solitaire, dating back to the 90s, has ads now.  I will refrain from additional opinion on that topic but it did make me think to myself: "Can I make my own version of this game?"

Fast forward an embarassing number of months later and I'm proud to say that yes, I was able to recreate the classic game in Unity with a fresh coat of paint and soundtrack to boot.

[Play Synthwave Solitaire on itch.io!](https://notaswe.itch.io/synthwave-solitaire)

# Development

This was without a doubt the largest programming-related project I have ever completed, so I figured I would post some notes about the development process.

# Engine Selection

Previously I have written very basic casino-style games leveraging the Pygame engine.  I have done several longform tutorials in other engines such as Godot and Unity as well; for this specific project, I figured I would use Unity because I was interested in leveling up my C# skills.  That said, I was very, very green in Unity/C# at the beginning of the project.

# The Card Class

In previous Pygame efforts (5 card Poker hand ranking, specifically) I realized that a traditional deck of cards is quite easily represented with a Card Class that gets instantiated n-number of times with varying information, to include card value, suit, and of course a Sprite representation.

When instantiating our 52-card deck, we can read from a dictionary/map that contains the information in order or out of order; it doesn't really matter.  What _does_ matter is that you employ some kind of [shuffling algorithm](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle) to the deck.

At this point, as far as Unity is concerned we have a Deck game object that has 52 instances of the Card class (one for each unique card).  Dealing the cards into their corresponding positions comes next.

# The Game Board

This part was a bit difficult for me to wrap my head around at first.  There are effectively seven "board columns" that cards get dealt to.  The index 0 board column receives the first dealt card face up, and then the remaining six columns receive the rest of the row face down.  We then shift one to the right, starting the process over but at index 1 board column, so on and so forth until we see the initial state:

![Cards In Position](/notablog/docs/assets/2025_05_25_deal.JPG "Cards In Position")

The remainder of the cards are then placed in a "stack" directly above the face up card in board column at index 0.  This is when the game officially starts.

# Implementing the Rules

The vast majority of development time was speant actually implementing the rules of Solitaire, which in essence are:

1. Cards can be moved and added as a "child" of a card with both a higher-by-one value and an opposite-color suit.
2. Kings are the only parent card that can move to an empty board column.
3. Aces are the only cards that can go into a blank Ace Zone.
4. Ace Zones behave differently than board columns.  They start as if they were numerically a "one" value and allow stacking only if the next card is valued one higher _and_ is of the same suit.
5. You can move "substacks" of cards if the receiving "parent" adheres to the above rules
6. The game is won when all Ace Zones are full

I will not go into the specific implementation details of the above but it resulted in multiple different methods and a ridiculously large Update() method.  Knowing what I know now, I would try to modularize the code a bit more if I were to start this endeavor by scratch.

# Graphical Considerations

In Synthwave Solitaire, I technically blend 3D with 2D as the background itself is a 3D Scene.  Being able to load one scene "within" another scene was what enabled this.  There are multiple cameras in use as well.

As for the background scene, the heavy lifting is being done by two separate shaders.  I had to create a sphere in Blender with significantly higher subdivision surfaces than the generic Unity sphere.  This was exported from Blender and opened in Unity, with the sun ray shader added to it.  The mountains were a low poly terrain, again with a shader added to it (albeit a grid shader this time).  I spent some time experimenting with procedural terrain generation but ultimately gave up on that effort.  I imagine I will revisit it again in the future because it is a really interesting concept - but tricky to implement from scratch.

An interesting dilemma with 2D games is the sorting layer itself.  Although Solitaire is a 2D game, the cards have a pseudo z-axis.  When you stack the cards, the lowest child card is actually rendered closer to the camera.  For the initial development of Synthwave Solitaire I kept incrementing the z-axis position of my Card instances; this ended up being a mistake that I later resolved by incrementing the sorting layer every time I interact with a card.  If this is not done properly, there will be instances where cards do not stack properly graphically.

# Final Thoughts

I wanted to continue adding features to Synthwave Solitaire but at the end of the day, I am happy with its current state.  You can actually beat the game and get a nice victory animation to boot.  As much fun as I have had with the development of this project, I am ready for my next endeavor in game development.  Until then, thank you for reading!

# Credits

I am by no means an artist and thankfully was able to leverage graphics and music provided for free from the following individuals:

- Cards - [Neon Orbis](https://neonorbis.itch.io/playing-cards)
- Skybox - [Dogmatic](https://assetstore.unity.com/packages/2d/textures-materials/sky/free-skyboxes-sci-fi-fantasy-184932)
  - Slightly modified coloration of a Skybox from this pack
- Soundtrack - Zach, my talented composer/producer friend
- UI - [Penzilla](https://penzilla.itch.io/basic-gui-bundle)
  - Graphics (UI Buttons) created by Penzilla Design; re-deal icon slightly modified from original