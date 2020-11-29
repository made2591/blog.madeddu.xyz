---
layout: post
title: "My first textual game"
date: 2018-01-02
polly: https://madeddu.xyz/mp3/textual-game.mp3
categories:
- coding
- python
tags:
- coding
- python
- vintage
- eighties
- game
---

### A beautiful childhood
I've never been a fan of videogames, neither as a child nor today: do not get me wrong, as many guys of my generation I also owned and played with the legendary __PlayStation 1__ (1995)[^p1] and __PlayStation 2__[^p2] by Sony and the __Nintendo Game Boy__[^gb]. It was fun, but not so much as playing to videogames born a few years before I was born.
In the 80s, there were not yet the powerful graphics chips that today can be found with a few hundred euros. How did the videogames work? It's simple: without graphics. There were many _textual games_ and in my opinion, for many vintage lovers, they would still be cool today. My generation has lost this advantage of literaly _building with fantasy the world around a game_. So, some days ago I found my self thinking back to Sheldoon Cooper that plays with the game Dungeon in the Big Bang Theory[^bbt], or to the kids of _Stranger Things_ by Netflix, or to my father, to whom I gave an original _Playstation 1_ with the first TombRaider(s), and I asked myself _how it would be possible today to play again one of those text games_ that were so popular in that decade...to the point of deciding to create one of my own.

<div class="img_container"><img src="https://i.imgur.com/dvGTQaE.png" style="width: 100%; marker-top: -10px;"/></div>

In this article I will talk about how to create a vintage 80s game! If you want to download the a technical preview of textual Tomb Raider right now, [here](http://github.com/made2591/tomb-raider) is the source code. If you want to find out how to build a game of the 80s - simply using Python[^ff] - read on!

### Recipe for a good game
Building a videogame, textual or not, is not the typical task for a computer scientists. Then, regardless of what your goal is and the complexity of the game you have in mind to create, you must first have a clear idea of _how you want_ your video game to work. In my honest opinion, a textual game must be as adventurous as possible, with all the characteristics of a classic modern rpg: a single player, items of different nature that affect the player and the environment in which he is immersed, player's life and/or statistics that interfere in performing certain actions, a simple and clear objective, even broken up into several simple tasks, one or more maps in which to hardly extricate themselves, possibly divided into levels too.

#### Ingredients
The first thing you need to build your adventure textual game is a character, usually described in the beginning of the game: this is done to help the player imagine its role in the adventure. I started with a modify version of the game description provided in the [wikiraider](https://www.wikiraider.com/index.php/Caves) page:

	Welcome to Tomb Raider I! You will interprete the role of Lara, a British archaeologist and treasure hunter, most commonly known for her discovery of several noted artefacts, including Excalibur, the fabled sword of King Arthur. You was hired by Jacqueline Natla to travel to the Andes mountain range, in Peru, in search of an artefact known as the Scion, which rests in the Tomb of Qualopec somewhere in the mountains. Accompanied by a guide you arrived at a huge pair of stone doors, the entrance to an ancient Incan civilisation. You climbed to the top of the doors and found a hidden switch that opens the doors. A pack of ferocious wolves suddenly attacked you from inside the cave. You leaped to the ground, killing the wolves in a barrage of pistol fire. However, you are too late, and your guide is dead in the snow.
	Alone, you headed into the caves in search for the village Vilcabamba.

What else?
- An amazing story: I decided to use the Tomb Raider story because I love Tomb Raider and Lara is one of my favourite characters;
- Previous adventure game experience: I think this could be ufeful in further step to build your textual game but not essential ;)

...and?
- A Python interpreter;
- A little bit of automata theory;

That's all! For those who are wondering why it is necessary to have theoretical foundations on automata, I answer "because this is not a guide to build textual game for painters but computer scientists, so let's add some nerd stuff to make things a little bit nerdy". Just kidding. Let's move one step forward, introducing game formalism and settings.

### Formalism and settings
In many years of study I learnt that _working with semantics_ is one of the most difficult task in NLP pipeline[^nlp]: if you don't believe me have a look to one of my [old repo](https://github.com/made2591/cognitive-system-postagger) on POS tagging with CYK[^cyk]. The point is simple: when you write a few line of codes, it's easy for a compiler or an interpreter to _understand_ what you mean. This is because the language you use is well formalised. The natural language is not well formalised: it's ambiguous, not deterministic, and works because human beings dialogue between each others with the help of thousand of other structures and methods to support the language. If we were to speak unambiguously without using any system of logical inference on the context, ignoring the gestures, tones of voice and facial nuances (actually, _understanding_ in the same way a compiler or a code interpreter would do), probably we would take 2 hours of conversation just to say hello to a friend of ours!

Over the years, I like to think also thanks to the incredible work of scientists like Chomsky[^nc], it was possible to build theories on the languages, on the grammars that support the more regular parts of them and, with the help of probabilistic calculation and with complicate deterministic systems, in the end Google translate was born. But there is a problem: _how_ could a game, whose input is textual, work in the early 80s? In other words, _how_ could a PC with a very little computational capacity (and without any theoretical support yet) correctly interpret sentences like ```jump into leaves``` or ```what is a rpbd``` and so on.
I didn't want to find in some webpage a pre-packaged answer to this question, because I like challenges: so I simply thought that input sentences in textual games are always simple... and this means that maybe it was possible to model textual analysis at a lower level: instead of working on _semantics_, I realized I could work on the _syntax_: in that moment I thought I could use a finite state automa.

#### The game's skeleton: finite state automa
Ok, I will use a finite state automa to build the game: how? Let's start with the basic concept: look at the picture.

<div class="img_container"><img src="https://i.imgur.com/yfGuw5G.png"  style="width: 250px; marker-top: -10px;"/></div>

In a textual game you are in a situation, you can do some actions, and change the situation. This can be implemented as a simple finite state automa, in which each possible scenario is a state, with the _right_ action that point to the next state in the story, and _epsilon_ moves that simply don't make any difference to your state. How do I build it? Starting from the actions: you first have to define which actions are allowed in your game. The original Tomb Raider is a free roaming game with several different kind of jump, move, climb and attack actions with many different interactions with items. If you want to create a more accurate rebuild, ok...you don't, so just start building a simple set of action.

For Lara, I add to my set of allowed actions ```walk```, ```run```, ```jump```, ```climb```, ```examine```, ```get```, ```use```, ```shot``` and of course ```save```[^sv]. After that, you can think about the states: each state is made of a description (printed in the cli of the player), than some items and - eventually - some npc that can interact with the main player. I thought about a state as simple dictionary of actions (keys) with other states as values (other keys) to which they bring to. Than, you can define items for a state with list of keywords, and so on.

Ok, we found out that finite state automa could be used to build your textual game. But how can you easly create and manipulate a finite state automa from a cli? Using dot? Of course, but an easier way came to my mind. You can use a new and innovative format: a JSON file.

#### The JSON game
Before definining a state, or ```step```, let's start with the ```player```: he usually has a ```name```, ```life```, the ```level``` and the ```step``` he reached, the ```items``` he __collected__ over the time, and some other generic properties useful for setup gaming speed and output.
{{< highlight json >}}
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
{{< / highlight >}}

The level struct could be defined as follow:
{{< highlight json >}}
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
                    "stepDescription": "There are
                    	 some footprints coming from the North"
                },
                "10": { [..] },
                [..]
            }
        }
    }
}
{{< / highlight >}}

As you can see, some actions don't have a step value, and this implies that those actions don't bring you anywhere / give you anything / etc. I wrote only a few transaction to the next state, but can you can add to the dictionary as many as you want.

__NOTE__: to make the game more playable, the player needs always a feedback, not only to gather new informations about the state, or even to understand more about items or interact with npc: to do that, I setup some default messages for _epsisol moves_: in the core game you can use this default message for each empty move typed by the player.

To summarize, what we said until now seems ok for moving action but...what about suffix and prefix, or action like ```examine the footprints```?

### Different type of action
Actually, actions to go ahead in the game (_aka_ available transaction between steps in the finite state automa), are not so simple as I argued. In fact, I splitted the allowed actions set in my game into two _game-agnostic_ set.

#### Move actions
The move action are simple: you can ```walk/run/jump``` in all direction (```straight/back/left/right```). To be more confortable, our core allow the player typing even the simple move action ```move``` or ```run```, mapping this to straight direction by default.

#### Interaction actions
The interaction actions are actions that _may_ require some additionatl information: why may? Because, I thought that could be useful to be able to ```examine``` both a room, to gather more information about the room itself, and a specific item in the scene, to get more information about the next steps to do. You can think about interation actions as a _jump to a secondary chain of steps in the level_ as shown in the schema below:

<div class="img_container"><img src="https://i.imgur.com/qDOrKbw.png"  style="width: 650px; marker-top: -10px;"/></div>

In this case, the chain is made of one single _step_: actually, it is not a real step, because if the _secondary story_ is made of a single assertion that need to help the player to enrich his knowledge about the state, you don't need a state. To handle this situation, I created a step, I print the step description, but I don't change the state, without jumping to the empty state. If the optional / secondary chain is made of more than one action - find a secret or optional item such as a medikit, you can create a real chain of effective states and let jump to the main story from the last state of the chain - of course if it is plausible in the history. In the example above, the information about footprints origin could help the player imagine an opponent npc to kill in the garden and check if the he has guns, or bullets, or whatever, to kill him.

### State with consequences
Some state has consequences: you might thinking "what do you mean?!". Ok, let say that you jump into a state like this:

	Three wolves are approaching in a hurry, they will attack you!

and you're ok, because a finite-state-automa based textual game is a game with a sort of stop motion graphic setup XD but remember...you are Lara Croft and those wolves are real. It is very likely that in the next state, or after a few, you will die... Fortunately, you are the programmer of the game, so the first problem you have to deal with is "how can I die in a finite state automa?"
- Option 1: in the next step you gonna die. This is simple to implement but wolves, or falling from a too high platform, or the insect sting don't hurt you with the same intensity in real world, and even if this solution will end in a "game over state" - in other words, you can't be wrong without die - _one-shot_ way;
- Option 2: you create _consequences_ in each steps;

Look at the code:
{{< highlight json >}}
{
	"player": { ... },
    "levels": {
        "cave": {
            "levelDescription": "Welcome [..]",
            "steps": {
                "0": {
                    "availableActions": { [..] },
                    "availableItems": ["footprints"],
                    "stepDescription": "The wolves are attacking you!"
                    "consequences": { "life" : -10 }
                },
                "10": { [..] },
                [..]
            }
        }
    }
}
{{< / highlight >}}

With a simple field you are done! Due to the consequences on some properties of the character, it will be easy to recreate the effect of attacks / falls / other types of interaction between the player and npc, or elements in the scene, describing the effects at each progression of state, through steps evolution in the automa. For example, with this mode, you can easly simulate a poisoning, as a consequence that for 10 steps decreases life according to a geometric series.

#### Target
The goal must always be clear: don't write too complicated descriptions to imagine, and always give a suggestion in each state about what the next move might be. A very important thing is the introduction of phrases that bind the various environments through which the player moves: for instance, sentences like

	There are some footprints coming from the North

are perfect to suggest without any consequences that maybe the North will be a direction involved in exploration (even if not exactly in the further step, but in some others in the story). Details can be left to secondary epsilon moves that add elements of the scene only if the player considers it necessary through actions such as ```examine```.

#### Non-player character (NPC)
To describe the behavior of the npc and the effects of the latter on the main player, you can use lists and dictionaries. For example, once the story was written, to begin with it is possible to create a list of all the entities that could interact with the main player during the evolution of the game. It is also possible to standardize some behaviors by creating distinct types of npc: for example, ferocious animals attacks provide damage to the player more or less always with the same intensity, or increasing intensity as the game progresses, or randomly in a given range. For the caves level of Tomb Raider, I defined the following npc(s):

	NPC_KEYWORDS = ["wolf", "wolves", "bear", "bears", "bat", "bats"]

Also arrows from walls can hurt you, so...try to model different type of npc if you want!

#### Actions
The actions are simple transactions to new states: I use items list to prohibit the player from, for instance, getting more than one medikit, and so on. The list of items in a scene (in a state) can also be useful to establish on which entities presented in the description it is possible to perform interaction actions (get, use, etc). You can defined as many actions as you want, as I said before. To help the player remember how to use the actions, I always make available in a help command that allows you to scroll in the style of man the list of actions available in the game, with instructions on the uses and effects of the same.

{{< highlight json >}}
    "actions": {
        "walk": {
            "usageMessage": "The WALK command lets Lara moving slow into the scenario. If Lara is not stucked by something in the scene, or there's any reason why she can not move, you will be always able to walk. You can move in a specific direction in the scene using the command WALK [STRAIGHT/DOWN/LEFT/RIGHT]. Note: if you type and execute WALK, you will WALK STRAIGHT by default."
        },
        "climb": {
            "usageMessage": "The CLIMB command lets Lara climb up/on something in the scene. You won't be always able to climb: Lara need something to climb up/on to complete this action. You can climb in the scene using the command CLIMB [UP/ON] [SOMETHING]. Note: if you type and execute CLIMB without specific element, Lara won't complete the action."
        },
        "examine": {
            "usageMessage": "The EXAMINE action is useful to help you explore the scene or a particular item in the scene. You can examine the scene using simply EXAMINE. If you try to examine an item you don't have in your pocket, the examine action will fail and nothing happens. You can examine an item using the command EXAMINE [ITEM_NAME] with the name of an item in the scene."
        },
        "get": {
            "usageMessage": "The GET action is used to get an item from the scene. If there are no items in the scene, nothing will be added to your pocket. You can get an item using the command GET [ITEM_NAME] with the name of an item in the scene."
        },
        "use": {
            "usageMessage": "The USE action is used to use an item you hold. If you can't use the item in the scene than the item will remain in your pocket. You can use an item using the command USE [ITEM_NAME] with the name of an item in the scene."
        },
        [..]
    },
{{< / highlight >}}

#### Items
The items interact with the player: eventually, they can also enable it for actions that are not available in a particular state. For example, it is not possible to light a flare to illuminate the scene if the player does not have a flare in the backpack, and so on. Another way to interact with items is to setup ```REGEX``` for actions that involve them, but I think it is more complicated. Write to me if you want more detail of these XD. ```REGEX``` could be useful in main to parse the sentece typed in input and loop through the automa.

#### Conclusion (first part)
Until now I wrote about how to create the logic behind a text game using finite-state automata: in the next paragraph I will talk about how I started working at Tomb Raider!

### Tomb Raider
Tomb Raider is a very difficult and complicated game to model: Lara can move through many actions, the set of items and weapons available is very extensive after a few hours of play, the enigmas are difficult and it is complicated to move through the rooms to reach levers, buttons, platforms and gear systems. Moreover, it is a non-textual game, so you need to replay the game and manually describe all the scenes from scratch in a new form. To help me with this difficult task, I wrote a simple Python script that does nothing more than generate a story _starting from a text_, with some basic default actions to progress through the states, and some tricks to set some of the properties described above.

#### A parser to find them all
Tomb Raider is one of those games so complex that it is not difficult to imagine that a lot of online solutions have been written: looking, I found in [wikirider](https://www.wikiraider.com/) a good example to start from - even if today I do not consider it the best source. In any case, what we need is a description of the game. For starters, I used the [caves](https://www.wikiraider.com/index.php/Walkthrough:Caves) walkthrough page as source. After the copy of the plaintext from the HTML page to a txt file, I used the following to create my draft version of states:

{{< highlight python >}}
# separate lines
with open("./levels/caves.txt", "r") as f:
	lines = f.readlines()
	steps = []
	for line in lines:
		steps += line.split(".")
# clean phase
steps = [x.strip().replace("\n","") for x in steps if x != "\n" or len(x.strip().replace("\n","")) > 1]
{{< / highlight >}}

The last line cleans the states list (list of sentences in my file) removing the empty lines and the titles. After that, you can create the steps for the level and also setup items / consequences looking npc and items in a (previously initialized) npc and items list of keywords. An example:

{{< highlight python >}}
index = 0
for step in steps:

	# create level structure
	GAME_STRUCTURE["levels"]["cave"]["steps"][index] = {
		"stepDescription" : step,
		"availableItems" : [],
		"availableActions" : {},
	}

	# insert items
	for keyword in ITEMS_KEYWORDS:

		# check if keyword of item appears in step description and it not already added to items list
		if keyword[:-1].lower().strip() in step.lower().strip() and keyword.lower().strip() not in GAME_STRUCTURE["levels"]["cave"]["steps"][index]["availableItems"]:

			# add to items
			GAME_STRUCTURE["levels"]["cave"]["steps"][index]["availableItems"].append(keyword.lower().strip())

	# create level structure
	GAME_STRUCTURE["levels"]["cave"]["steps"][index]["availableActions"] = {

		"walk" : str(index+STEP_OFFSET),
		"walk straight" : str(index+STEP_OFFSET),
		[..]

	index += INDEX_OFFSET
{{< / highlight >}}

If you have a look to my previous examples, I setup state identifier with an ```INDEX_OFFSET``` (value = 10). Why? Because after the automatic phase, you need to add / modify each steps to create a real gaming experience, and in the main loop through each state could be usefull to have keys that can be ordered but also the ability to insert new states (primary or secondary) without breaking the original order.

#### Formalize situations
The next step in creating a safe and easy way to expand and test the entire game-experience, is to formalize the several situations ih which the player could come across: the discovery of a medikit, the attack by an NPC. Every situation is repeatable by abstracting to the automaton that describes its evolution, and these automata can then be populated with distinct descriptions in order to be reused when necessary during the description of each level. And that's how I now have to formalize separate automata and update my guide ... XD

#### Conclusion (second part)
If you enjoy this article, please share it! [Here](http://github.com/made2591/tomb-raider) the source code of my work-in-progress Tomb Raider textual version.

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