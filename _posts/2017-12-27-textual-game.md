---
title: "My first textual game"
tags: [coding, vintage, eighties, game, golang]
---

### A beautiful childhood
I've never been a fan of videogames, neither as a child nor today: do not get me wrong, as many guys of my generation I also owned and played with the legendary __PlayStation 1__ (1995)[^p1] and __PlayStation 2__[^p2] by Sony and the __Nintendo Game Boy__[^gb]. It was fun, but not so much as playing to videogames born a few years before I was born. 
In the 80s, there were not yet the powerful graphics chips that today can be found with a few hundred euros. How did the videogames work? It's simple: without graphics. There were many _textual games_ and in my opinion, for many vintage lovers, they would still be cool today. My generation has lost this advantage of literaly _building with fantasy the world around a game_. So, some days ago I found my self thinking back to Sheldoon Cooper that plays with the game Dungeon in the Big Bang Theory[^bbt], or to the kids of _Stranger Things_ by Netflix, or to my father, to whom I gave an original _Playstation 1_ with the first TombRaider(s), and I asked myself _how it would be possible today to play again one of those text games_ that were so popular in that decade...to the point of deciding to create one of my own.

<p align="center"><img src="https://res.cloudinary.com/lmn/image/upload/e_sharpen:100/f_auto,fl_lossy,q_auto/v1/gameskinnyc/c/o/l/colossalcaveadventure1-100023877-orig-99278.jpg" style="width: 100%; marker-top: -10px;"/></p>

In this article I will talk about how to create a vintage 80s game! If you want to download the first version of Horizon right away, [here]() is the source code. If you want to find out how to build a game of the 80s - simply using Python[^ff] - read on!

### Recipe for a good game
Building a videogame, textual or not, is not the typical task for a computer scientists. Then, regardless of what your goal is and the complexity of the game you have in mind to create, you must first have a clear idea of how you want your video game to work. In my honest opinion, a textual game must be as adventurous as possible, with all the characteristics of a classic modern rpg: a single player, items of different nature that affect the player and the environment in which he is immersed, player's life and/or statistics that interfere in performing certain actions, a simple and clear objective, even broken up into several simple tasks, one or more maps in which to hardly extricate themselves, possibly divided into levels too.

#### Ingredients
The first thing you need to build your adventure textual game is a character, usually described in the beginning of the game. This is done to help the player imagine its role in the adventure: I started with a modify version of the game description provided in the [wikiraider](https://www.wikiraider.com/index.php/Caves) page:

	Welcome to Tomb Raider I. You will interprete the role of Lara, a British archaeologist and treasure hunter, most commonly known for her discovery of several noted artefacts, including Excalibur, the fabled sword of King Arthur. You was hired by Jacqueline Natla to travel to the Andes mountain range, in Peru, in search of an artefact known as the Scion, which rests in the Tomb of Qualopec somewhere in the mountains. Accompanied by a guide you arrived at a huge pair of stone doors, the entrance to an ancient Incan civilisation. You climbed to the top of the doors and found a hidden switch that opens the doors. A pack of ferocious wolves suddenly attacked you from inside the cave. You leaped to the ground, killing the wolves in a barrage of pistol fire. However, you are too late, and your guide is dead in the snow. 
	Alone, you headed into the caves in search for the village Vilcabamba.

What else?
- Of course an amazing story: I decided to use the Tomb Raider story because I love Tomb Raider and Lara is one of my favourite characters;
- Previous adventure game experience: I think this could be ufeful in further step to build your textual game;

...and?
- A python interpreter;
- A little bit of automata theory;

That's all! For those who are wondering why it is necessary to have theoretical foundations on automata, I answer "because this is not a guide to build textual game for painters but...computer scientists, so let's add some nerd stuff to make things a little bit nerdy". Just kidding. Let's move one step forward, introducing game formalism and settings.

### Formalism and settings
In many years of study I learnt that _working with semantics_ is one of the most difficult task in NLP pipeline[^nlp]: if you don't believe me have a look to one of my [old repo](https://github.com/made2591/cognitive-system-postagger) on POS tagging with CYK[^cyk]. The point is simple: when you write a few line of codes, it's easy for a compiler or an interpreter to _understand_ what you mean. This is because the language you use is well formalised. The natural language is not well formalised: it's ambiguous, not deterministic, and works because human beings dialogue with the help of thousand of other structures and methods to support the language! If we were to speak unambiguously without using any system of logical inference on the context, ignoring the gestures, tones of voice and facial nuances (actually, _understanding_ in the same way a compiler or a code interpreter would do), probably we would take 2 hours of conversation just to say hello to a friend of ours.

Over the years, thanks to the incredible work of Chomsky[^nc], it was possible to build theories on the languages, on the grammars that support the more regular parts and, with the help of probabilistic calculation and with complicate deterministic systems, in the end something good was pulled out. But there is a problem: in the 80s how could a game, whose input was textual, work? In other words, how could a PC with a very little computational capacity (and without any theoretical support yet) correctly interpret sentences like ```jump into leaves``` or ```what is a rpbd```
and so on. 

I did not want to find an answer to this question, because I like challenges so I thought: input sentences are simple... maybe it is possible to model this analysis at a lower level: instead of working on _semantics_, I could work on the _syntax_. I could use a finite state automa.

#### The game's skeleton: finite state automa
Ok, I will use a finite state automa to build the game: how? Let's start with the basic concept: look at the picture.

<p align="center"><img src="https://image.ibb.co/gs7zBb/automa.png" alt="perceptron" style="width: 250px; marker-top: -10px;"/></p>

In a textual game you are in a situation, you can do some actions, and change the situation. This can be implemented as a simple finite state automa, in which each possible scenario is a state, with the _right_ action that point to the next state in the story, and _epsilon_ moves that simply don't make any difference to your state. How do I build it? Starting from the actions: you first have to define which actions are allowed in your game. The original Tomb Raider is a free roaming game with several different kind of jump, move, climb and attack action with many different interactions with items. If you want to create a more accurate rebuild, ok...you don't, so just start building a simple set of action.

For Lara, I add to my set of allowed actions ```walk```, ```run```, ```jump```, ```climb```, ```examine```, ```get```, ```use```, ```shot``` and of course ```save```[^sv]. After that, you can think about the states: each state is made of a description (printed in the cli of the player), than some items and - eventually - some npc that can interact with the main player. I thought about a state as simple dictionary of actions (keys) with other states as values (other keys) to which they bring to. Than, you can define items for a state with list of keywords, and so on.

Ok, we found out that finite state automa could be used to build your textual game. But how can you easly create and manipulate a finite state automa from a cli? Using dot? Of course, but an easier way that came to my mind: you can use a new and innovative format: a JSON file. 

#### The JSON game
Before definining a state, or ```step```, let's start with the ```player```: he usually has a ```name```, ```life```, the ```level``` and the ```step``` he reached, the ```items``` he __collected__ over the time, and some other generic properties useful for setup gaming speed and output.
{% highlight json %}
{
	"player": {
        "columns": 80, 
        "items": [], 
        "level": "cave", 
        "life": 100, 
        "name": "made2591", 
        "step": 0, 
        "velocity": "debug"
    }
}
{% endhighlight %}

The level struct could be defined as follow:
{% highlight json %}
{
	"player": { ... },
    "levels": {
        "cave": {
            "levelDescription": "Welcome to Tomb Raider I! 
				 You will interprete the role of Lara [..]", 
            "steps": {
                "0": {
                    "availableActions": {
                        "examine": "", 
                        "get": "", 
                        "run": "10", 
                        "use": "", 
                        "walk": "10", 
                        [..]
                    }, 
                    "availableItems": [
                        "footprints"
                    ], 
                    "stepDescription": "Footprints"
                }, 
                "1",
            }
        }
    }
}
{% endhighlight %}

As you can see, the empty move don't bring you anywhere. I wrote only a few transaction to the next state, but can you can add to the dictionary as many as you want. __NOTE__: to make the game more playable, the player needs always a feedback, not only to gather new informations about the state, or even to understand 

#### Target

#### Non-player character (NPC)

#### Actions

#### Items

### Tomb Raider

#### Instructions



Thank you everybody for reading!

[^p1]: Yes, it's incredible. Playstation 1 goes back to 90s. The console was released on 3 December 1994 in Japan, 9 September 1995 in North America, 29 September 1995 in Europe, and for 15 November 1995 in Australia - [source and more](https://en.wikipedia.org/wiki/PlayStation_(console))
[^p2]: The Playstation 2 belongs to millennials OMG - [source and more](https://en.wikipedia.org/wiki/PlayStation_2)
[^gb]: The first version of Game Boy is older than PS1 - [source and more](https://en.wikipedia.org/wiki/Game_Boy)
[^bbt]: The Big Bang Theory, [The Irish Pub Formulation](http://www.imdb.com/title/tt1632243/), Season 4 Episode 6
[^ff]: I would like to work to more modern version with Docker and Spring Boot!
[^nlp]: Read more about [Natural language processing](https://en.wikipedia.org/wiki/Natural_language_processing)
[^cyk]: Ok, this could be hard for those who are new to NLP problems...be careful: [CYK](https://en.wikipedia.org/wiki/CYK_algorithm)
[^nc]: Pioner in language theory [Wikipedia](https://en.wikipedia.org/wiki/Noam_Chomsky)
[^sv]: To save the game in the original Tomb Raider (PS1 version) you need some crystal you encounter during the levels.