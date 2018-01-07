---
layout: post
title:  "Quinto"
klipse: true
---

I played an old board game called Quinto when I was visiting a friend this past Thanksgiving. Afterward, I developed a strange fixation on the game and wrote [a program](http://jrheard.com/quinto/) that lets you play Quinto against a computer opponent. I'm going to show you the game and talk about the tools I used to build it.

I'll start by teaching you how to play the game - don't worry, there are just like three rules. If you'd like to skip ahead, [here's the game](http://jrheard.com/quinto/) and [here's the box](quinto.jpg).

How It's Played
==============

Quinto is basically like Scrabble, except with numbers instead of letters. In Scrabble, your goal is to place several tiles in a row or column, and have them spell a word; you get extra points if your freshly placed tiles contact several pre-existing words. Quinto's the same thing, except that instead of trying to make words, you're trying to make a run of tiles **whose sum is a multiple of five**. For example, this move would earn you 20 points:

7 8 5

This move is invalid, though, because these tiles sum to 17, which is not a multiple of five.

7
7
3

TODO explain scoring

5 3 2
    3
	5

if you placed the 3 5, you get 10 points

5 3 2
5 7 3
	5

if you placed the 5 7, you'd get 10 + 10 + 15 points = 35

Now you know how to play Quinto! The game ends when you run out of tiles; whoever has the highest score wins.

But Wait!
---------

There's one more rule: you can never make a move that would cause there to be a run of more than five tiles in a row. For instance, this move is invalid:

5 3 2 5 5 0
          3
		  2

Placing that 0 would cause there to be six tiles in an unbroken row, which breaks the rules; so you can't make that move.

It turns out that this last rule makes the game really hard to play. When you're deep into a game and there are a ton of tiles on the board, it takes _all_ of my brainpower to try to look at the tiles in my hand, look back at the board, and feverishly check to see if placing these three tiles over _here_ would - no, that's not a multiple of five. Hm, maybe over here! Yes, perfect! Except - oh no, I can't put a tile down on _that_ space, because that would break the no-more-than-five-tiles-in-a-row rule!

This drove me just completely nuts. I was like: if you can't make a move on a space, the board should light that space up in red! But of course the board couldn't do that, because it's just a dumb piece of cardboard. So I decided to write a computer program that would light up invalid cells in red, and playable cells in green, like this:

8 7 5 3 2
        8

That turned out to be so easy that it was actually kind of unfulfilling. Then I found myself wondering: gee, how would you go about writing an AI that plays the game? Once I was done with that, I hooked things up so that a human can play against the AI, and now [I'm done](http://jrheard.com/quinto/). I can't remember the last time I actually _finished_ a side project. It feels pretty good!

Now I'd like to tell you about the tools I used to build this game.

Tools
=====

Clojure
-------
learned clojure in 2011, been using it for side projects since then
all the data structures are immutable by default, there's a strong culture of functional programming in the language's community, but you can still easily perform side effects whenever you want to
the language sits on top of java so you get a library ecosystem for free, and the libraries that the clojure community has created a really amazingly good

clojure is a fantastic language for writing programs that transform and filter data[1]
[1] this is all programs

clojurescript
-------------
but i actually wrote my program in clojurescript, which is a version of clojure that compiles to javascript. cljs sounded like a goofy idea to me when i first heard about it, but i've been using it for years now and i completely love it, because in addition to all the benefits you get with clojure, cljs has two additional great things going for it:

* cljs programs are almost always written to be run inside of a web browser, which means that if you already know how to write HTML and CSS, using cljs gives your clojure program a UI for free!
* because cljs programs can easily be run inside of a web browser, you can just upload your program somewhere and send your friend a link to it, and they can use your program without having to install anything!

on top of that, you can also use any javascript library as well as most clj libraries, and again the libraries that the clojurescript community has written are really really good.

clj/s is truly a sweet spot for writing turn-based games, because you get to focus on happily writing simple pure functions that express the game's business logic, and your UI is just a pure function of your "game state", and clojure's "atom" abstraction makes it easy for you to manage that state confidently.

reagent
-------
would have tried to write this myself if it didn't already exist, saved me a lot of trouble, also is _extremely great_

intellij/cursive
---------------
i used vim for years to write clojure, but i figured i'd try cursive because a personal license is free, and i loved it, never looked back.
i use the vim plugin and paredit

repl, comment blocks - maybe a gif about this?

figwheel
-------
somehow express haumann's thing about how usually you have to make a change to your code, then restart your program and navigate it back to the same state that you were looking at before you made your change, and how figwheel solves that problem

and also just feels extremely extremely good to use

spec
----
spec rules, is an incredibly great addition to the language; before spec people used a great community library called schema, but having an official way to do this is a fantastic thing for the language
:rets aren't checked by instrumentation, orchestra solves this

specter
-------
explain what the tool is and give examples of how i used it
mainly for selects, didn't use transformations
mention that writing custom navigators was really really easy (although my code turned out to be hideous for performance reasons)
thank nathan for the help

color picking
-------------
I don't have a good tool for this, and that really bothers me. The colors I picked are awful, but all the other combinations I tried were even worse. What would you have done if you were building this game and had to pick colors for UI elements? Do you have any advice?

techniques
==========

dev diary
---------
context dump - any time i notice that i'm not sure what code to write next, i force myself to narrate my thoughts into this text file
immense value, primarily in two categories:
* forces me to clearly articulate my thought process (this extremely helps get to a better solution, faster!)
* historical record - makes it easy for me to jump back into the project after some time has passed. i just look at the last page of my dev diary, and now i know what i was working on when i put this down.

i haven't completely figured this out - i sometimes run into issues when i'm working on multiple projects, or on multiple branches within the same project; i'm confident that i'll end up with a working system, and when i do i'll write about it

is-grid-valid? fn
-----------------
read about this somewhere in coders at work, can't remember what chapter
whenever you're working with a novel data structure (mine isn't at all novel but has certain constraints), write a validation function and have an assert somewhere that calls it
this was really really really useful, went off about 15-16 times over the course of development
found three main categories of bug:
* the starting grid i had manually hard-coded didn't actually sum to a multiple of five
* a bug in the AI code was causing the computer to make an invalid move
* a bug in a core quinto.grid function broke validation, so that valid grids were being reported as invalid
each time one of these bugs was introduced, i was immediately notified because an assert failed, and i was able to immediately fix the bug; without that assert, days could have passed during which the program's logic was quietly incorrect
unit tests would have surfaced these categories of bugs too, but this program is small and i didn't feel like writing unit tests for it

at the end of development i realized that this was running way way more often than it needed to, so i removed that call and my perf sped up 10x, whoops

that's it!
----------
if you'd like to get started with clojure, consider XYZ resources (brave and true?)



TODO refer to https://lambdaisland.com/blog/29-12-2017-the-bare-minimum-clojure-mayonnaise somewhere
or maybe don't


TODO if it turns out to be a good story, talk about how i tracked down the heisenbug by writing a program that played the game's UI




appendix

quinto origin story
brunner