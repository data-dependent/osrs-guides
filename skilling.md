# A Guide For Tick Manipulation Mechanics In Skilling
Throughout this guide, we'll introduce skilling mechanics and tick-by-tick descriptions of optimal skilling methods. This text is not intended as a guide to perform one particular method, but rather as a guide to broadly understand skilling. 

Few of the ideas presented here are due to the writers. Thanks to Bea5, Drew, Fraser, GeChallengeM, Henke, Illysial, Jamal, Jukebox Romeo, Julia, Nechs, Tannerdino, and others for their explanations and helpful discussions. They're always great to chat with, but, in particular, feel free to reach out to _vireo (data_dep)#3975_ or _Port Khazard#2280_ to chat about the contents of this guide.

### Table of Contents

 - [**Ticks via Henke's model**](#tick-manipulation-i-ticks-via-henkes-model)
    - [Client input](#client-input)
    - [NPC and player turns](#npc-and-player-turns)
       - [Stalls](#stalls)
       - [The queue](#the-queue)
       - [Timers](#timers)
       - [The area queue](#the-area-queue)
       - [Interactions](#interactions-with-objectsitems-and-npcsplayers)
 - [**The Skilling Tick**](#tick-manipulation-ii-the-skilling-tick)
    - [Early examples](#early-examples)
    - [Inventory actions](#inventory-actions)
       - [Eating](#eating)
    - [Flinching](#flinching)
    - [Review of specialized mechanics](#review-of-specialized-mechanics)
 - [**Stalls**](#tick-manipulation-iii-stalls)
    - [Stalls after movement](#stalls-after-movement)
    - [Stalls on successful rolls](#stalls-on-successful-rolls)
    - [Stalls in cooking](#stalls-in-cooking)
 - [**Some problems**](#some-problems)

## Tick Manipulation I: Ticks via Henke's Model

The shortest unit of time on the Old School Runescape server is one game tick, or just _tick_ for short. This means that describing the state of the game for each tick provides a complete and perfect description. While this is true, it's important that the description is read out at the same place during each tick. This is because actions are executed each tick, so the game state incrementally changes as time progresses within a tick. Henke's model provides a powerful framework to understand the order of execution of actions within each tick.

````
Server tick:
	for each player:
		Process client input

	for each npc:
		Stalls end
		Timers
		Queue
		Interaction with objects/items
		Movement
		Interactions with players/npcs

	for each player:
		Stalls end
		# Close interface if strong command queued
		Queue
		Timers
		Area queue
		Interaction with objects/items
		Movement
		Interaction with players/npcs
		# Close interface if trying to log
````

At the highest level of complexity, Henke's model asserts that commands within a tick happen in three ordered groups: client input, then npc turns, then player turns. The npc turns happen in order of npc ID, and the player turns happen in order of player ID, or pid. In the remainder of this section, we describe each of these groups and their constituent elements.

### Client input

Clicks on one tick are not processed until client input on the next tick. This can sometimes create an appearance of lag. We show this in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/OEadZs3.gif" alt='Opening bank interface' width=500>

On the tick that we clicked the bank, we also right clicked the range. Had the bank interface appeared instantly, it would not have been possible for us to right click the range.

At most ten commands will execute in client input during any tick. For example, if twenty-eight unclean herbs are clicked on then an adjacent and reachable tile is clicked on in the same tick, then ten herbs will be cleaned on the next tick, then ten herbs will be cleaned on the next tick, then the final eight herbs will be cleaned and movement will happen on the next tick. Note, rarely only the last ten commands will be executed on the tick after the clicks.

### NPC And player turns

After all client input is processed, all non player characters (or npcs) will take their turns. Following that, all players will take their turns. An immediate consequence of this ordering is that PVMers typically do not have to be mindful of their player ID, whereas PVPers do. This is since in PVP our enemy could have their turn either before or after ours.

Within each turn, the terms which appear (such as 'timer' and 'queue ') are categories of commands, and the ordering of the these categories denotes the ordering of the execution of the corresponding commands within a turn. We will mostly focus here on player turns, however do notice that this ordering is not the same for npcs and players.

#### Stalls

When our character is stalled, our client input and our turn is almost completely blocked: for the most part, our timers don't go off, our queue doesn't evaluate, we can't interact with anything, we can't move, and no client input is accepted. An example of a stall comes from teleporting. When we click a teleport in the normal spellbook, a stall starts in client input on the next tick. The stall ends four ticks later, when our coordinates change to our destination. We'll see interesting skilling examples of stalls in the [**Stalls**](#tick-manipulation-iii-stalls) section.

#### The queue

The queue is a driver of much behavior in game. In this section, we'll first focus on commands unrelated to skilling to initially develop our understanding. Commands in the queue evaluate so that the first that come in are the first that come out. 

A basic example of a queued command is hitsplats. On a turn that a player or npc interacts with an enemy to attack, a command to deal damage is put into the enemy's queue, to be evaluated the next time the enemy has a turn. Notice the asymmetry in this example due to the turn ordering: npcs deal damage to players on the same tick they attack, whereas players deal damage to npcs on the tick after they attack. This can be seen by looking carefully at Henke's model, which we do below.

<div style="text-align:center"><img src="https://i.imgur.com/BRUE6wn.png" alt='NPC attacks player' width=300>

In this first image, we consider what happens when the NPC attacks. The attack itself happens during the NPC's turn, and then our character receives the damage during our turn in the queue. 

<div style="text-align:center"><img src="https://i.imgur.com/i3g7kXw.png" alt='Player attacks NPC' width=300>

In this second image, we consider what happens when we attack. Since our turn is late within a tick, when we attack, the NPC's next turn will be on the following tick. Since NPCs also recieve damage during their turn from their queue, the damage appears on the NPC on the next tick.

A clear illustration of the position of the queue as being in our turn is provided by trying to kill ourselves with a locator orb and a zamorak brew. Both of these do damage based on our hitpoints at the moment our click on them is processed; however, zamorak brews do their damage in client input, while locator orbs queue their damage. At 11 hitpoints, if we click on a zamorak brew then a locator orb on the same tick, the following clip happens.

<div style="text-align:center"><img src="https://i.imgur.com/serp8Qp.gif" alt='Zammy brew then locator orb' width=500>

Below we provide an alternative view of the clip by writing out what's occurring on the server in text. All of the actions are executed within one tick.

During client input:
 - Zammy brew checks current hp (11), and damage (1) is calculated,
 - Zammy brew deals 1 damage,
 - Locator orb checks current hp (11-1=10), and damage (9) is calculated, then
 - Locator orb sends 9 damage to queue.

During the queue on our turn:
 - Locator orb deals 9 damage.

The situation is different if we perform the clicks in the opposite order, which we see in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/HitPpiN.gif" alt='Locator orb then zammy brew' width=500>

Below, we again provide an alternative view of the gif by writing out what's occurring on the server in text. After our health hits zero, our death is put into our queue to be evaluated on the next tick.

During client input:
 - Locator orb checks current hp (11), and damage (10) is calculated,
 - Locator orb sends 10 damage command to our queue,
 - Zammy brew checks current hp (11), and damage (1) is calculated, then
 - Zammy brew deals 1 damage.

During the queue on our turn:
 - Locator orb deals 10 damage, and our health hits 0.

There are three types of commands: weak, normal, and strong. Weak commands get deleted by any actions that interrupts. Since having a strong command in our queue will interrupt us every tick, weak commands get deleted when there is a strong command queued. Note that interfaces also get closed by interruptions. An example of a command being deleted appears often in wintertodt: most fletches there are weak commands, while the damage is a strong command. Almost all client input also interrupts, making a good quick test for whether a command is weak in the queue is whether rearranging items in our inventory deletes it.

Commands in skilling are most commonly weak, because actions not based on interactions tend to happen as weak commands in the queue. For example, making herb tar uses a weak command in our queue. After being afk for some time, if we use tar on an herb, a weak command will be put into our queue to complete the make in three ticks. If we don't do any actions which interrupt, the herb tar will be made. Recall that we can quickly test that making herb tar is indeed a weak command since rearranging our inventory cancels the action. Other examples are most repetitive actions, such as in fletching, potion making, cooking, wintertodt brazier feeding, smithing, and crafting. For most of these, the first action occurs in client input if the first action happens on the tick after the click, then later actions occur from the queue.

Herb picking in farming provides an interesting and rare example of a queued normal command in skilling. Upon a first interaction with an herb spot, a command is added to our normal queue to pick an herb (after a 1t stall). Because normal commands cannot be interrupted like weak commands, this provides a 1t window to do nearly any other action, including restarting interacting with the herb spot.

<div style="text-align:center"><img src="https://i.imgur.com/92uKlfC.gif" alt='1.5t herb picking' width=500>

In the clip, there are two sequences of herb picks happening every 3 ticks. Further relevant herb picking mechanics which will be written in the tick-by-tick description below are that later picks are from weak commands and that there is always a 1t stall between the queued command popping and receiving the herb.
 - **Tick 1**: During client input, start interacting with herb spot. Then, during the player's turn in interactions, a command to pick (A) is added to normal queue for next tick.
 - **Tick 2**: During client input, restart interacting with herb spot. During player's turn in queue, a stall starts.
 - **Tick 3**: During player's turn when stall ends, recieve an herb (A) and weak queue a pick command for 2t. Then, in interactions, a command to pick (B) is added to normal queue for next tick.
 - **Tick 4**: During player's turn in queue, a stall starts.
 - **Tick 5**: During player's turn when stall ends, recieve an herb (B) and weak queue a pick command  for 2t. Next, in queue, a stall starts.
 - **Tick 6**: During player's turn when stalls ends, recieve an herb (A) and weak queue a pick command  for 2t.
 - **Tick 7**: During player's turn in queue, start a stall.
 - **Tick 8**: During player's turn when stalls ends, recieve an herb (B) and weak queue a pick command  for 2t. During player's turn in queue, start a stall.
 - **Tick 9**: During player's turn when stalls ends, recieve an herb (A) and weak queue a pick command  for 2t.

#### Timers

During the category timers, commands which are to be executed periodically are executed. For example, cannons do all of their work once a tick during timers. Prayer drain also occurs during timers. Poison "goes off" during timers and puts a hitsplat in our queue, which is evaluated on the next tick.

#### The area queue

The area queue is similar but separate from the queue. It still evaluates on a first in, first out basis, but it only takes in commands related to area effects based on being on particular tiles. When our character is on certain tiles, called _area tiles_, commands are put into our area queue. When an interface is open or our character is stalled, movement is blocked when our area queue is not empty: therefore a helpful test for whether a tile is an area tile is to walk across it with an interface open. Note that this test is not sufficient to establish that a tile is an area tile, since walking with an interface open is blocked when some commands are in our queue and by some timers.

The most common sorts of area tiles, which are also the most easily locatable/identifiable are
 - map chunk lines (easy to get off google),
 - multi-combat lines (easy to get off google or check yourself), and
 - bank-zone borders.

An interesting consequence of this for skilling is that some tiles should be avoided when fletching broads, as shown below.

<div style="text-align:center"><img src="https://i.imgur.com/axwNuLc.gif" alt="Port's skilling area tile example" width=500>

The [clip](https://streamable.com/5f1zir) shows a line of area tiles on fossil island which are frequently crossed while hunting the Herbiboar. Starting to fletch on these tiles should be avoided because they prevent movement if you cross over them with an interface open, for as long as the interface is open.

#### Interactions with objects/items and npcs/players

Interactions are one of the most useful categories for skilling. They're separated into two types: interactions with object or items and interactions with npcs or players. In between the two, movement occurs, which has many noticeable consequences in game. One of them is that interaction with a banker (an npc) happens sooner than interaction with a bank booth (an object) when moving up to them, as shown in the clips below.

<div style="text-align:center"><img src="https://i.imgur.com/WKVnLz9.gif" alt='Interactions with banker' width=500>

<div style="text-align:center"><img src="https://i.imgur.com/TsnL1GA.gif" alt='Interactions with bank booth' width=500>

In the first clip, we move then interact with the banker in the same tick. In the second clip, we move on a tick then interact with the bank booth on the following tick. The same behavior can be seen when moving up to a tree (an object) to woodcut or when moving up to a fishing spot (an npc) to fish.

Interactions can be used to produce an optimal method to cook. Below we describe an optimal cooking method which works for any food item which can produce more than one cooked product, such as raw beef, raw karambwan, and raw bear meat. Note, an optimal cooking method for other food is described later in [Stalls in cooking](#stalls-in-cooking).

<div style="text-align:center"><img src="https://i.imgur.com/5JiU7Mp.gif" alt='1t cooking' width=500>

The method works even in f2p where no second option explicitly appears. We now describe in text the commands shown in the clip.
 - **Tick 1**: In client input, we start interacting with the fire. During our turn, in interactions with objects, a choice interface opens.
 - **Tick 2**: In client input, our choice is processed and the cook happens. Next in client input, we start interacting with the fire again. During our turn, in interactions with objects, a choice interface opens.
 - **Tick 3**: In client input, our choice is processed and the cook happens. Next in client input, we start interacting with the fire again. During our turn, in interactions with objects, a choice interface opens.

Notice that the first raw beef is cooked before the next raw beef begins to be processed. Each tick, we restart interacting with the fire since otherwise later cooks would happen more slowly from our queue.

## Tick Manipulation II: The Skilling Tick

Most tick manipulation in skilling amounts to carefully controlling the so-called skilling tick. There's two relevant variables for this: One is a `global tick counter` that starts at 0 on a server reboot and increments by one every tick before any player's client input and another is a player-specific variable called the `skilling tick` that specifies a value of the global tick counter where a skilling action can occur.

Sometimes people refer to the skilling tick minus the global tick counter as the _skilling timer_, which can be conceptually convenient since it lets us not be explicit about the value of the global tick counter. Note that the skilling timer is unrelated to "timers" in Henke's model. The skilling timer counts down by one every tick and the skilling action occurs when the skilling timer "goes off" and hits 0. We will refer to the skilling timer most of the times that we mention setting the value of the skilling tick: for convenience, we sometimes write that we are delaying the skilling timer by _k+1_ in place of writing that we are setting the skilling timer to _k_. This is helpful because there are _k+1_ ticks between when the timer is set and when it goes off; for example, harpoon fishing a shark takes 6 ticks, but only sets the skilling timer to 5 on the first interaction. 

In most skilling examples, which skilling action occurs when the skilling timer goes off is determined through what we're interacting with. We can only interact with one entity (object, item, npc, or player) at a time. 

A first example of the use of the skilling tick can be found in woodcutting, which standardly is a 4 tick action. Let's set the stage: suppose that the global tick counter is 998 and the skilling tick is 900. This roughly means that 98 ticks have passed since we've completed our last skilling action. 
 - On global tick 998, let's say we click a reachable tree while there's one tile between our character and the tree. 
 - Then, on global tick 999, our click is processed in client input where our interaction entity is set to the tree; later in the tick, during our turn, we move one tile to be adjacent to the tree. 
 - On global tick 1000, during our turn, we interact with the tree, and, since our skilling tick is less than the global tick counter, our skilling tick is set to 1003. (Put differently, this sets the skilling timer to 3 or delays the skilling timer by 4.) 
 - On global ticks 1001 and 1002, we interact with the tree during our turn largely with no effect. 
 - On global tick 1003, which is the value of the skilling tick, our interaction with the tree during our turn produces a roll. If the roll succeeds, we get a log and there's a possibility of the tree falling.

### Early examples

While mining, we roll for an ore against any rock that we're interacting with during the skilling tick. This means in part that stopping interacting with a rock then restarting interacting with a rock has the same effect as continually interacting with a rock. While mining iron in the mining guild, we can use this idea to buy an extra tick for reaction time, as shown below.

<div style="text-align:center"><img src="https://i.imgur.com/AxUdwnW.gif" alt="Bea's mining guild iron" width=500>

Using a dragon pickaxe and crystal pickaxe provides a chance of delaying the skilling timer by two ticks instead of the usual three ticks. We control for this by _always_ moving to another rock after two ticks. On the next tick, we then move back to the previous rock if it isn't depleted. We also drop ore in client input before starting to interact with a rock to keep space in our inventory.

The skilling tick is just a number, so it doesn't know what kind of action set the skilling tick. In the iron mining example, the skilling tick both got set from an iron rock and was used to mine an iron rock; however, this consistency was not needed. We can also set the skilling tick using one action and then complete a distinct action during the skilling tick. This strategy is behind almost every optimal skilling method in the game. A first example of this is from setting the skilling timer using woodcutting (which is 4 ticks) then completing a barbarian fishing action (which is by default 5 ticks), as shown in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/FL2K5q7.gif" alt='Tree fishing' width=500>

Below we describe in text the actions in the clip.
-  **Tick 1**: The skilling timer decrements to -1. During client input, we start interacting with the tree. During our turn, we interact with the tree which sets our skilling timer to 3.
-  **Tick 2**: The skilling timer decrements to 2. During client input, we start interacting with the fishing spot.
-  **Tick 3**: The skilling timer decrements to 1. 
-  **Tick 4**: The skilling timer decrements to 0. During our turn, in interactions with npcs, we get a roll for a fish.

In doing this method, we click on the tree on the same tick as the roll from fishing: on the next tick, **Tick 1**, this makes us interact with the tree when the skilling timer gets delayed. This method, known as _tree fishing_, was optimal near the start of Old School, before more ways to set the skilling tick were known.

Combat does also use the skilling tick, but it uses the skilling tick in a different way than skills like fishing, mining, and woodcutting. While we are interacting with an npc or player, we attack them whenever the skilling timer is nonpositive. On the same tick as we attack, our skilling timer gets to set to our weapon attack speed. This behavior is different than in fishing, mining, and woodcutting, where the skilling timer gets set on the tick after the skilling action, and we can use it to speed up fishing even further.

<div style="text-align:center"><img src="https://i.imgur.com/95cO4aX.gif" alt="Jamal's 3t fishing with darts" width=500>

In the clip, we attack an alt with a dart while our attack options are set to rapid, and then start interacting with the fishing spot. Below we describe this in more detail.
 - **Tick 1**: The skilling timer decrements to -1. During client input, we start interacting with our alt. During our turn, in interactions with players, we attack our alt with rapid darts, setting our skilling timer to 2. 
 - **Tick 2**: The skilling timer decrements to 1. During client input, we start interacting with the fishing spot.
 - **Tick 3**: The skilling timer decrements to 0. During our turn, in interactions with npcs, we get a roll for a fish.

In this method, we stop interacting with the fishing spot on **Tick 1** so that our skilling tick instead gets set from the darts.

### Inventory actions

A major breakthrough in skilling occurred when the community found inventory items which could delay the skilling timer without being consumed. This was done around the time of the Skilling Cups with the discovery that making herb tar, which takes three ticks, uses the skilling tick. With this, nearly all of fishing, mining, and woodcutting actions could now be done every three ticks. In the following examples, notice that the inventory actions we use to delay the skilling timer do so during client input, which makes some of the methods possible.

An example of 3t skilling with barbarian fishing is below, where we use a knife on a teak log rather than an herb on swamp tar, although these are essentially equivalent from the perspective of the skilling tick.

<div style="text-align:center"><img src="https://i.imgur.com/pAPlYsy.gif" alt='3t Barbarian fishing with knife-log' width=500>

Below we describe in text the actions in the clip.
 - **Tick 1**: The skilling timer decrements to -1. During client input, we start making a teak stock, which removes our interaction with the fishing spot, sets our skilling timer to 2, and puts a weak command in our queue to make a teak stock.
 - **Tick 2**: The skilling timer decrements to 1. During client input, we start interacting with the fishing spot. This interruption deletes the teak stock command from our queue.
 - **Tick 3**: The skilling timer decrements to 0. During our turn, in interactions with npcs, we get a roll for a fish.

This rhythm repeats: notice that on **Tick 1** we crucially interrupted our fishing with a knife-log to set the skilling timer using the knife log rather than the fishing spot.

The knife-log used above set the skilling timer after the skilling timer had gone off. Most actions which can change the skilling tick similarly have this _after the skilling timer has gone off_ condition. One way to make sense of this is through an example: while interacting with a fishing spot every tick, the skilling tick change condition is passed every tick, so to make getting rolls possible, the skilling tick change must only happen _after_ the roll, which occurs on the skilling tick.

#### Eating

Not all actions which change the skilling tick have this _after the skilling timer has gone off_ condition. In fact, eating most food will add three ticks to the skilling tick, regardless of the value of the skilling timer. We'll case such food items _3t foods_. 

When the skilling timer is positive, we can directly see the effect of the additive delay from eating. While attacking an enemy, our skilling timer will always be nonnegative, therefore making eating a 3t food always slow us down by three ticks.

<div style="text-align:center"><img src="https://i.imgur.com/f7ZU5p2.gif" alt="Combat delay from eating" width=500>

Below we describe in text the actions in the clip.
 - **Tick 1**: During our turn, we attack the guard. This sets our skilling timer to 3, due to our 4 tick weapon.
 - **Tick 2**: The skilling timer decrements to 2.
 - **Tick 3**: The skilling timer decrements to 1.  During client input, we eat a swordfish, making our skilling timer be 1+3=4.
 - **Tick 4**: The skilling timer decrements to 3. During client input, we start interacting with the guard.
 - **Tick 5**: The skilling timer decrements to 2.
 - **Tick 6**: The skilling timer decrements to 1.
 - **Tick 7**: The skilling timer decrements to 0. During our turn, in interaction with npcs, we attack the guard. This sets our skilling timer to 3.

If the skilling tick is in the past, eating a 3t food won't necessarily bring the skilling tick into the future. Therefore, in contrast to eating while attacking, our attacks won't be slowed down if (after being idle) we eat before starting to attack.

<div style="text-align:center"><img src="https://i.imgur.com/87dd8IH.gif" alt='No delay when eating before combat' width=500>

Below we describe in text the actions in the clip.
 - **Tick 1**: The skilling timer is some large negative number, say -999. During client input, we eat a swordfish, making our skilling timer be -999+3=-996. During client input, we start interacting with the guard. During our turn, in interaction with npcs, we attack the guard, setting our skilling timer to 3.
 - **Tick 2**: The skilling timer decrements to 2.
 - **Tick 3**: The skilling timer decrements to 1.
 - **Tick 4**: The skilling timer decrements to 0. During our turn, in interaction with npcs, we attack the guard, setting our skilling timer to 3.

Eating food will bring the skilling tick into the future when the skilling timer is a small enough negative number. For instance, when the skilling timer is -1, eating a 3t food such as roe or caviar moves the skilling timer to 2. Since this is the same effect as from a knife-log, sometimes this is referred to as eating having the ability to "continue cycles". An example is shown below.

<div style="text-align:center"><img src="https://i.imgur.com/WayNBjJ.gif" alt='Cut-eat 3t barb fishing' width=500>

This method is mechanically similar to the previous 3t fishing with a knife-log example, with some additions. Below we describe in text the actions in the clip.
 - **Tick 1**: The skilling timer decrements to -1. During client input, we eat a roe or caviar, which removes our interaction with the fishing spot and adds three to our skilling timer to make it -1+3=2. Next in client input, we process a fish into a roe or caviar. Next in client input, we start interacting with the fishing spot.
 - **Tick 2**: The skilling timer decrements to 1.
 - **Tick 3**: The skilling timer decrements to 0. During our turn, in interactions with npcs, we get a roll for a fish.

In this version of 3t fishing, on **Tick 1**, we also cut a fish in client input to produce more food to eat and some cooking experience. We also start interacting with the fishing spot on **Tick 1** rather than on **Tick 2**, even though either would work. An interesting variant of this method includes one more action: we pick up a fish on **Tick 1** (by interacting with a ground fish) then start interacting with the fishing spot on **Tick 2**. The result of this is that we have more fish to cut, giving more cooking experience.

Some food items add two ticks to the skilling tick rather than three ticks. A typical example of this kind of food is karambwans. This shorter delay can be used to our advantage to skill faster. We can't using karambwans to 2t skill since they can only be eaten at most once every three ticks. However, we can use karambwans to extend 3t fishing into 2.5t fishing, where we alternate between getting a roll in three ticks and a roll in two ticks.

<div style="text-align:center"><img src="https://i.imgur.com/hAIRd8S.gif" alt='2.5t barb fishing' width=500>

Below we describe in text the actions in the clip.
 - **Tick 1**: The skilling timer decrements to -1. During client input, we start making a teak stock, which removes our interaction with the fishing spot, sets our skilling timer to 2, and puts a weak command in our queue to make a teak stock.
 - **Tick 2**: The skilling timer decrements to 1. During client input, we start interacting with the fishing spot. This interruption deletes the teak stock command from our queue.
 - **Tick 3**: The skilling timer decrements to 0. During our turn, in interactions with npcs, we get a roll for a fish.
 - **Tick 4**: The skilling timer decrements to -1. During client input, we eat a karambwan which sets the skilling timer to -1+2=1. Next in client input, we start interacting with the fishing spot.
 - **Tick 5**: The skilling timer decrements to 0.  During our turn, in interactions with npcs, we get a roll for a fish.

A cleverly timed second karambwan eat can be put into the 2.5t method to make skilling even faster. To do the method, we will eat the second karambwan two ticks after a roll, rather than one tick after like for the first karambwan. Unfortunately, this method cannot be used at barbarian fishing, which has a mechanic where the first interaction with the fishing spot will never produce a roll.

Here's the method shown for 2.33t willows. The method is called that because we get 3 rolls every 7 ticks, making the average number of ticks per roll equal to 2 1/3.

<div style="text-align:center"><img src='https://i.imgur.com/Nw2M8ty.gif' src="https://i.imgur.com/dG0kl78.gif" alt="Port Khazard's 2.33t method" width=500>

Below we describe in text the actions in the clip.
 - **Tick 1**: During client input, we start making a teak stock, which removes our interaction with the tree, sets our skilling timer to 2, and puts a weak command in our queue to make a teak stock.
 - **Tick 2**: The skilling timer decrements to 1. During client input, we start interacting with the tree. This interruption deletes the teak stock command from our queue.
 - **Tick 3**: The skilling timer decrements to 0. During our turn, in interactions with objects, we get a roll for a log.
 - **Tick 4**: The skilling timer decrements to -1. During client input, we eat a karambwan which sets the skilling timer to -1+2=1. Next in client input, we start interacting with the tree.
 - **Tick 5**: The skilling timer decrements to 0.  During our turn, in interactions with objects, we get a roll for a log.
 - **Tick 6**: The skilling timer decrements to -1. During client input, we remove our interaction with the tree.
 - **Tick 7**: The skilling timer decrements to -2. During client input, we eat a karambwan which delays our sk illing timer to -2+2=0. Next in client input, we start interacting with the tree. During our turn, in interaction with objects, we get a roll for a log.

### Flinching

Being attacked can affect the skilling tick. In more detail, while our auto-retaliate is on, we are not interacting with anything, and our skilling tick is in the past, being attacked will set our skilling tick to half of our attack speed, rounded down. This is called being _flinched_. This mechanic was likely designed for use in combat, but it is used in several optimal skilling methods.

With a 2 tick or 3 tick weapon, being flinched will make the skilling tick be the next tick. While being attacked every other tick, this can make the skilling tick be set to be the next tick every other tick, as shown in the example below.

<div style="text-align:center"><img src="https://i.imgur.com/BQR6DDl.gif" alt='2t prif teaks' width=500>

This method is 2t teaks, and was the optimal method for woodcutting experience and the pet before fossil island introduced a spot which allows for 1.5t teaks. The method involves a two tick rhythm, described in text below.
 - **Tick 1**: The skilling timer decrements to -1. In client input, we remove our interaction with the tree. During a rabbits turn, it attacks us and puts damage into our queue. During our turn, when our queue evalutes, we recieve the damage, which sets our skilling timer to 1 and makes us start interacting with the rabbit.
 - **Tick 2**: The skilling timer decrements to 0. In client input,  we start interacting with the tree. During our turn, in interactions with objects, we to get a roll a log. 

The interaction with the tree at the beginning of **Tick 1** can be removed in a number of ways, including via a no-movement path (as in the clip), log dropping, or dart fletching. Notice that we are at no point interacting with a rabbit during any skilling tick, which keeps us from shooting a dart.

Essentially the same method is also done for swordfish harpooning, as shown below.

<div style="text-align:center"><img src="https://i.imgur.com/5QyQZVc.gif" alt='2t pisc swordfish harpooning' width=500>

Interestingly, in this variant of the method, we don't need to make any clicks to cancel our interaction with the fishing spot. This is because harpoon fishing spots (like most others) force you to stop interacting with them after one tick before the skilling timer has gone off (except for when the skilling timer is 4, which allows for afk fishing). This force stop of our interaction with the fishing spot is the same reason why the rhythm for 3t swordfish fishing is different than the rhythm for 3t barbarian fishing: we still have to start interacting with the fishing spot on the skilling tick.

### Review of specialized mechanics

In this section, we'll review and collate the specialized mechanics which were introduced above. These are mechanics beyond just keeping track of the skilling tick which we should keep in mind while thinking about skilling methods.

The skilling timer delay can happen either after the action or on the action. Fishing, mining, and woodcutting actions happen on the skilling tick, and then the next interaction (while the skilling timer is negative) sets the skilling timer. However, combat and firemaking actions happen when the skilling timer is nonpositive, and the skilling timer delay happens on the same tick as the action. This difference is intrinsically notable but can also be combined to produce a method like PVP world 3t fishing, shown previously.

Inventory actions like making herb tar, wood stocks (via knife-logging), kebbit claws, or battlestaves (via celastrus bark) don't complete when we start them on the skilling tick. This is because the check to delay their skilling timer is whether the skilling timer is _nonpositive_. This is unlike fishing, mining, or woodcutting, where the check to delay their skilling timer is whether the skilling timer is _negative_. Another example of unusual behavior can be seen in snow gathering: there, the skilling timer is delayed by three whenever the skilling timer is less than two.

The first interaction can have different behavior than later interactions. We saw this with barbarian fishing, where we could not get rolls during the first interaction with a fishing spot, even if the first interaction is during the skilling tick. Other examples of this behavior appear at fly fishing, pike fishing, and boulder mining in volcanic mine.

In fishing, sometimes when the skilling timer is nonnegative we are forced to stop interacting with the fishing spot after one tick. We saw this during 2t swordfish, where this mechanic made us not need to click off the fishing spot. Barbarian fishing, fly fishing, pike fishing do not have this mechanic at all, while lava eel fishing has a variant of it.

Most new skilling activities in the game unfortunately do not use the skilling tick. In fishing, the examples are karambwan, infernal eel, minnow, and sacred eel fishing. In mining, examples include mining for dense essence blocks in Zeah.

This section has focused on the skilling tick, but there are other "named" ticks which function in much the same way. Particularly prominent examples are the alchemy tick (which several normal spellbook spells use), the eating tick (which control which ticks we can eat most non-karambwan food), the karambwan eating tick, the pot sipping tick, and the bone bury tick. For an example of delaying these ticks, note that eating a karambwan will block eating, potion sipping, and karambwan eating for three ticks. This is because eating a karambwan delays all of the eating, the pot sipping, and the karambwan eating ticks. However, a typical non-karambwan food item will only delay the eating tick, so karambwans and potions can still be immediately consumed. The reason why karambwans can't be used to 2t at barbarian fishing is, in more detail, because karambwans delay the karambwan eating timer by three ticks, despite delaying the skilling timer by only two ticks.

## Tick Manipulation III: Stalls

Stalls appear often during skilling, sometimes to block increased experience rates and sometimes to improve experience rates.

There are of course immediate examples of unhelpful stalls. Agility obstacles stall our character while we cross them. This means that no amount of ingenuity will ever speed up crossing an obstacle, as we're blocked during crossing. Another example is picking up a box trap, which produces a three tick stall. Being stalled here means that a trap can be reset in at fewest three ticks, which is only possible if we can set up a box trap on the turn that our stall ends. However, this isn't possible, so the fastest a trap can be reset in practice is in four ticks.

Another example where a stall slows us down is below. However, here, an understanding of mechanics saves us a tick compared to a naive method. Notice that our bank interface opens on the same tick that a stall ends.

<div style="text-align:center"><img src="https://i.imgur.com/us896Tz.gif" alt='Interface opening after stall' width=500>

The spell crucially did not remove our interaction with the bank chest, which saves a tick over just casting the spell then clicking the bank on the tick the stall ends. Below we describe in text the actions in the clip.
- **Tick 1**: In client input, we start interacting with the bank chest. Next in client input, we cast the superglass make spell, which starts a stall and does not remove our interaction.
- **Tick 2**:
- **Tick 3**:
- **Tick 4**: During our turn, the stall ends. Next during our turn, in interaction with objects, the bank interface opens.

Note that clicking the bank a tick before the stall ends will not be processed since the stall ends after client input.

### Stalls after movement

Some objects produce a one tick stall before fully executing our interaction with them after moving in the previous tick. For example, changing levels via most ladders in game is slowed down by one tick due to this stall. There are primarily two places where this stall appears in skilling: while woodcutting player-farmed trees or mining rocks. For example, when afk mining, this stall slows down experience when moving in between rocks: it forces us to take five ticks until we receive a roll instead of the usual four ticks (one for movement, three for the skilling timer delay from mining).

This stall can be used to help us by providing up to two rolls on a skilling tick, rather than just one roll. Below we show a clip of 1.5t teaks on fossil island, which makes use of this stall.

<div style="text-align:center"><img src="https://i.imgur.com/LoHe2Ae.gif" alt='fossil island 1.5t teaks' width=500>

Below we describe in text the actions in the clip.
 - **Tick 1**: In client input, we start making herb tar, which removes our interaction with the tree, sets our skilling timer to 2, and puts a weak command in our queue to make a teak stock. Next in client input, we path to move, which deletes the teak stock command. During our turn, in movement, we move.
 - **Tick 2**: In client input, we start interacting the tree. During our turn, in interaction with objects, a stall starts.
 - **Tick 3**: During our turn, the stall ends. Next during our turn, in interaction with objects, we get a roll for a log.

Significantly, our interaction with the tree was paused when the stall began, so our interaction completes when the stall ends. This finishing of our interaction from **Tick 2** is the source of the additional roll. 

Unfortunately, this method can easily go wrong. Below, we consider two examples of common mistakes while attempting to make use of this stall. The [clips](https://streamable.com/xry1fh) are at the essence mine, where each roll is guaranteed to be successful. If our character interacts with the essence rock two ticks after moving rather than one, the stall won't happen and we will lose out on one roll, as seen below.

<div style="text-align:center"><img src="https://i.imgur.com/RVhTXra.gif" alt="Interacting late during double roll method" width=500>

Below we describe in text the actions in the clip.
- **Tick 1**: During client input, we use a knife on a teak log, which sets the skilling timer to 2 and puts a weak command in our queue to make a teak stock. Next during client input, we  set a destination tile, which removes the teak stock command from our queue. During our turn, in movement, we move.
- **Tick 2**: The skilling timer decrements to 1.
- **Tick 3**: The skilling timer decrements to 0. During client input, we start interacting with the Rune Essence. During our turn, in interaction with objects, we receive an essence.

Further, if we move on the tick _after_ delaying the skilling timer by three, our character will be stalled when they would have interacted with the essence rock during the skilling tick, giving no rolls at all.

<div style="text-align:center"><img src="https://i.imgur.com/WUD8dGj.gif" alt="Moving late during double roll method" width=500>

Below we describe in text the actions in the clip.
- **Tick 1**: During client input, we use a knife on a teak log, which sets the skilling timer to 2 and puts a weak command in our queue to make a teak stock. 
- **Tick 2**: The skilling timer decrements to 1. During client input, we set a destination tile, which removes the teak stock command from our queue. During our turn, in movement, we move.
- **Tick 3**: The skilling timer decrements to 0. During client input, we start interacting with the Rune Essence. During our turn, in interaction with objects, a stall starts.

On **Tick 4**, when the stall ends, the skilling timer isn't 0 so we don't receive any rolls.

The same method based on double rolls at 1.5t teaks is also commonly used while mining sandstone and granite in the quarry. There we still can get two rolls, but the second roll only offers us a second chance for when the first roll fails. This is perhaps the only example in game of tick manipulation being used to increase the probability of successfully gathering a resource in a tick.

<div style="text-align:center"><img src="https://i.imgur.com/qTXZQgk.gif" alt='3 tick 4 granite' width=500>

### Stalls on successful rolls

Most mining rocks produce a one tick stall after successfully rolling a resource. However, iron, granite, sandstone, limestone, volcanic sulphur, and rune essence do not have this stall. Some rocks which do not deplete only have a one tick stall for the first resource gathered per click to interact; examples include dayaelt, volcanic ash, and pay-dirt mining. 

The effects of this stall are most well known for gem mining, where 3t inventory actions are used to produce a resource roughly every four ticks instead of the usual three. 

<div style="text-align:center"><img src="https://i.imgur.com/QfhZkYo.gif" alt='4t gem mining' width=500>

Below we describe in text the actions in the clip. Suppose that the roll we recieved was from the interaction on **Tick 3**, not from the interaction in **Tick 2** which completes when the stall ends.
- **Tick 1**: During client input, we use a kebbit claw on a d'hide vambrace, which sets the skilling timer to 2 and puts a weak command in our queue to make a kebbit vambrace. Next during client input, we  set a destination tile, which removes the teak stock command from our queue. During our turn, in movement, we move.
- **Tick 2**: The skilling timer decrements to 1. During client input, we start interacting with the gem rock. During our turn, in interaction with objects, a stall starts.
- **Tick 3**: The skilling timer decrements to 0. During our turn, at the beginning, the stall ends and we unsuccessfully roll for a gem. Next during our turn, in interaction with objects, we successfully roll for a gem and a stall starts.
- **Tick 4**: The skilling timer decrements to -1. During our turn, at the beginning, the stall ends and we receive experience, receive a gem, and the gem rock depletes.

### Stalls in cooking

Another example of an unhelpful stall can be found when cooking food besides raw beef, raw bear meat, and raw karambwans. The 1t cooking method doesn't work for other food items due to a stall which forces the first cook of these raw food items to take four ticks. 

However, since the stall only appears when there's more than one of the same raw food in our inventory, it's still possible to accelerate cooking these other food items by directly circumventing this problem. Indeed, we can cook faster by only cooking with one raw food in our inventory at a time. We now consider two examples of this. First, we consider a case where we can make more raw food using inventory actions.

<div style="text-align:center"><img src="https://i.imgur.com/8WfMumW.gif" alt='1t combine-cooking' width=500>

This shows a step involved in cooking for the mess hall in Zeah. Amazingly, we are still able to 1t cook. Below we describe in text the actions in the clip.
- **Tick 1**: In client input, we combine cheese and incomplete pizza to produce an uncooked pizza in our inventory. Next in client input, we start interacting with the stove. During our turn, in interaction with objects, we cook the pizza.

Second, we consider a case where we need to do some interactions to get more raw food.

<div style="text-align:center"><img src="https://i.imgur.com/3netQ5Y.gif" alt='2t bank cooking' width=500>

Below we describe in text the actions in the clip.
- **Tick 1**: In client input, we withdraw a shark from our bank. Next in client input, we start interacting with the stove. During our turn, in interaction with objects, we cook the shark.
- **Tick 2**: In client input, we start interacting with the bank. During our turn, in interaction with objects, an interface to our bank opens.

Every so often, in client input on **Tick 1**, we first deposit our inventory. Needing to interact to get more raw sharks forces shark cooking to be at least 2 ticks, one tick for interacting with something to get a shark and one tick for interacting with the stove.

## Some problems

Here we put in some fun problems to help solidify the previous material.

   1. Most of the methods presented were shown with a particular choice of click timing, despite there being other clicks which yield equivalent methods. Which methods have this wiggle room?
   2. What's the most number of times that bones to bananas can be cast in 29 ticks? <!-- one per tick, have to pick up and cast in same tick and also drop when needed -->
   3. In free-to-play, there are no 2t weapons which players can equip. Describe a method on a PVP world using only flinching to mine rune essence with the fewest average ticks per roll. Is this the fastest possible method? <!-- long bow on rapid, alt attacking every 4 ticks; no, eat + move -->
   4. In free-to-play, a ground item (snow) can be used to 3t skill. The best method for woodcutting experience off of PVP worlds involves cutting willows with snow while being attacked by a rat, which has a 4t attack speed. Describe how to combine snow gathering and flinching to produce a method with the lowest average number of ticks per roll. <!-- 2.66t -->
   5. Continuing from problem 4, what's the best method when the attacking npc has other attack speeds, such as 2 ticks, 3 ticks, or 5 ticks? <!-- 2t, 3t, 2.5t -->
   6. Describe trap resetting in chinchompa hunting using Henke's model.
   7. Tree fishing is an interesting method in part because it doesn't involve any special items or attacking. Are there any other skilling actions which can have their average number of ticks per roll reduced without any special items or attacking? <!-- addy pick 2t ess or teak/mahog 2t and clicking net spot every 5 ticks and dropping chin trap and picking up to start 4t trap resetting and cook eat fishing -->
   8. After describing cut-eat barbarian fishing, an extended method was briefly described involving additionally picking up fish from the ground to cut for more cooking experience. Is it possible to pick up two fish from the ground within the three tick rhythm of the method? <!-- no, the interaction immediately before and on the xp drop tick are used by the fishing spot and so there's only one interaction slot open -->
   9. Is it possible to use a dragon/crystal harpoon spec during 2t swordfish without losing ticks? Is it possible to get a lamp from a genie without losing ticks? (Note, while moving between spots doesn't count.) <!-- Yes, weapon speed doesn't matter when not being hit and then change back happens in client input before being hit, but we cannot talk to genie without losing ticks since that uses up an interaction -->
   10. The volcanic mine community posts recommendations on when to eat karambwans or sip brews while mining. They suggest to click the karambwan or brew and then the boulder on the tick after an experience drop while mining. How many ticks are lost using this approach? Describe another approach which loses no ticks. <!-- 2 for karam, 3-4 for brew if pick 2t's; click on xp drop as with most food -->
   11. Describe two distinct methods for 3t swordfish, where there's a roll once every three ticks. At most one method should involve attacking. <!-- 1) we have a 2t or 3t wep and alt has a 3t wep, 2) we click fish spot a tick later than for 3t barb -->
   12. Without varrock armour 4 or the mining cape, is it possible to gather two runite ore in one tick while mining a runite rock? <!-- maybe with mining gloves, but, no, the successful roll stall blocks it -->
   13. During 4t gem mining with a rune pickaxe, if moving between rocks every four ticks, explain why failing to receive a gem at one rock will force the next rock to also not yield a gem. Describe a more efficient variant of this gem mining method. <!-- the skilling timer delay on tick 4 (since not stalled when unsuccessful) makes it so we're "moving late" like in 3t4g, a fix is to ground click in case not stalled or use addy pick for fix or just try to herbtar+move on tick3 and tick4 -->
   14. The method for 2t sharks we presented relied crucially on being adjacent to a bank. Without a bank, describe a way to cook a shark once every two ticks. When next to a bank, which method would you prefer? <!-- drop cooking, which is worse cuz it takes time to bank and drop -->
   15. Is it possible to cook one shark and one swordfish in the same tick? What about two sharks in the same tick? <!-- a: yes by waiting 3t after interface opens then selecting option+using fire on fire (ie interacting) in same tick, 15b: Same methods works here (2 shark, 1 sword in inventory), but naively using both on stove makes only last one cook (since cook happens in interaction) -->
   16. In [2014](https://www.youtube.com/watch?v=OjusYIftpZw), [2015](https://www.youtube.com/watch?v=Jn-vf4RMwww), and [early 2017](https://www.youtube.com/watch?v=XpCMUJzCL0M), the skilling community put out videos displaying their best methods. Which mechanics were discovered in between those years? Have any new mechanics been discovered since early 2017?
   17. What do you wish would have been presented differently in this guide?
