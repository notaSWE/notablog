---
title: "Solitaire But Make It Synthwave"
date: 2025-05-25
---
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

At this point, as far as Unity is concerned we have a Deck game object that has 52 instances of the Card class (one for each unique card).  At that point in time, dealing the cards into their corresponding positions comes next.

# The Game Board

This part was a bit difficult for me to wrap my head around at first.  There are effectively seven "board columns" that cards get dealt to.  The index 0 board column receives the first dealt card face up, and then the remaining six columns receive the rest of the row face down.  We then shift one to the right, starting the process over but at index 1 board column, so on and so forth until we see the initial state:

![Cards In Position](/notablog/docs/assets/2025_05_25_deal.JPG "Cards In Position")

The remainder of the cards are then placed in a "stack" directly above the face up card in board column at index 0.  This is when the game officially starts.