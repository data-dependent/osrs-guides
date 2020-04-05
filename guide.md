# A Guide For Tick Manipulation Mechanics In Skilling
Throughout this guide, we'll introduce skilling mechanics and tick-by-tick descriptions of optimal skilling methods. This text is not intended as a guide to perform one particular method, but rather as a guide to broadly understand skilling. 

​	Few of the ideas presented here are due to the writer. Thanks to Bea5, Drew, Fraser, GeChallengeM, Henke, Jamal, Julia, Nechs, Tannerdino, and all of The Summit for their explanations and helpful discussions. The aforementioned people are always great to talk to, but, in particular, data_dependent#3975 always has his DMs open to chat.

##### Table of Contents

[Ticks via Henke's model](#tick-manipulation-i-ticks-via-henkes-model): Intro to intratick behavior

[The skilling tick](#tick-manipulation-ii-the-skilling-tick): Overview of the driver of skilling actions

[Stalls](#tick-manipulation-iii-stalls): Overview of action blockers in skilling

[The queue](#tick-manipulation-iv-the-queue): Overview of use of queue for skilling

[Movement and pathing](#tick-manipulation-v-movement-and-pathing): Overview of relevant movement mechanics during skilling

## Tick Manipulation I: Ticks via Henke's Model

The shortest unit of time in Old School Runescape is one game tick, or just _tick_ for short. This means that describing the state of the game for each tick provides a complete and perfect description.

While it is true that providing a description of the game once a tick provides all of the information about the game, it's important that the description is read out at the same place during each tick. This is because actions are executed each tick, so the game state incrementally changes as the time progresses within a tick. Henke's model provides a powerful framework to understand this order of execution of actions within each tick. 

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

At the highest level of complexity, Henke's model asserts that: first, _client input_ is evaluated for each player; second, each nonplayer character, or npc, takes its _turn_ in order of npc ID; last, each player takes its _turn_ in order of player ID, or pid.

### Client input

Clicks on one tick are not processed until client input on the next tick. This can sometimes create an appearance of lag. We show this in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/OEadZs3.gif" alt='Opening bank interface' width=500>

On the tick that we clicked the bank, we also right clicked the range. Had the bank interface appeared instantly, it would not have been possible for us to right click the range.

At most ten commands will execute in client input during any tick. For example, if twenty-eight unclean herbs are clicked on then an adjacent and reachable tile is clicked on in the same tick, then ten herbs will be cleaned on the next tick, then ten herbs will be cleaned on the next tick, then the final eight herbs will be cleaned and movement will happen on the next tick.

### NPC And player turns

After client input is processed, all non player characters (or npcs) will take their turns. Following that, all players will take their turns. An immediate consequence of this ordering is that PVMers typically do not have to be mindful of their player ID, whereas PVPers do, since in PVP our enemy could have their turn either before or after ours.

An interesting way to take advantage of the ordering of client input and the turns is through using mithril seeds to attack an npc while the npc is unable to attack back, such as in the clip below. Note that mithril seeds move the player during client input.

<div style="text-align:center"><img src="https://i.imgur.com/7FEB1fG.gif" alt='Mith seed to avoid damage' width=500>

The method used in the clip is to: On Tick 1, during our character's turn, both move from under the Kalphite Queen, or just KQ, and attack KQ. Then, on Tick 2, during our client input, move to under KQ via mithril seeds. The player is never in range of KQ during one of her turns, since her turn on Tick 1 is before our actions and her turn on Tick 2 is after our action.

### Elements of turns

Within each turn, the terms which appear (such as 'timer' and 'queue ') are categories of commands, and the ordering of the these categories denotes the ordering of the execution of the corresponding commands within a turn. At the beginning of both npcs and players turns, stalls end; these will be explained in a later section. We will mostly focus here on the player turns, however do notice that this ordering is not the same for npcs and players.

#### The queue

The queue is a driver of much behavior in game. In this section, we'll focus on commands unrelated to skilling to initially develop our understanding. Commands in the queue evaluate so that the first that come in are the first that come out. A basic example of a queued command is hitsplats. On a turn that a player or npc interacts with an enemy to attack, a command to deal damage is put in the enemy's queue, to be evaluated the next time the enemy has a turn. Notice the asymmetry in this example due to the turn ordering: npcs deal damage to players on the same tick they attack, whereas players deal damage to npcs on the tick after they attack.

There are three types of commands: weak, normal, and strong. Commands higher in this typology delete lower commands; that is, weak commands get deleted from our queue if there are normal or strong commands queued, and normal commands get deleted if there are strong commands queued. An example of a command being deleted appears often in the nightmare zone. A locator orb applies damage as a normal command in queue that gets deleted when there are incoming projectiles, which are strong commands. Almost all client input also deletes weak commands in the queue, so a  good quick test for whether a command is weak in the queue is whether rearranging items in our inventory deletes it. 

A clear illustration of the position of the queue as being in our turn is provided by trying to kill ourselves with a locator orb and a zamorak brew. Both of these do damage based on our hitpoints at the moment our click on them is processed; however, zamorak brews do their damage in client input, while locator orbs queue their damage. At 11 hitpoints, if we click on a zamorak brew then a locator orb on the same tick, the following clip happens.

<div style="text-align:center"><img src="https://i.imgur.com/serp8Qp.gif" alt='Zammy brew then locator orb' width=500>

The zamorak brew click is processed in client input: first our current hitpoints (11) is checked and damage (1) is calculated, and we immediately incur that 1 damage. Then, the locator orb click is processed in client input: first our current hitpoints (10) is checked and damage (9) is calculated, then a hitsplat of 9 damage is put in our queue, which we then incur during our turn later in the tick.

The situation is different if we perform the clicks in the opposite order, which we see in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/HitPpiN.gif" alt='Locator orb then zammy brew' width=500>

The locator orb click is processed in client input: first our current hitpoints (11) is checked and damage (10) is calculated, then a hitsplat of 10 is put in our queue. Then, the zamorak brew click is processed in client input: first our current hitpoints (11) is checked and and damage (1) is calculated, and we immediately incur that 1 damage. Later, during our turn when the queue evaluates, we incur the 10 damage from the locator orb and die. Our death is put into our queue to be evaluated on the following tick.

#### Timers

During the category timers, commands which are to be executed periodically are executed. For example, cannons do all of their work once a tick during timers. Prayer drain also occurs during timers. Poison "goes off" during timers and puts a hitsplat in our queue, which is evaluated later in the same tick.

#### The area queue

The area queue is similar but separate from the queue. It still evaluates in a first in, first out basis, but it only takes in commands related to area effects based on standing on particular tiles. An example is from the wilderness obelisk, shown below.

<div style="text-align:center"><img src="https://i.imgur.com/iHaEtB3.gif" alt='Wilderness obelisk teleport' width=500>

Eleven ticks after interacting with the obelisk (which is not shown), the lights shut off and a command evaluates from our area queue that puts a command to teleport in our queue. On the following tick, which is the next time the queue is evaluated, the teleport is executed. This tick gap between the area queue and the queue is the reason why both we can move when we do in the clip and we can see the lights shut off before we teleport.

Another example of this one tick delay due to the ordering of the queue and area queue occurs when walking on a poisoned tile, such as while fighting Xarpus during the Theater of Blood. A clip is shown below, where we take a hitsplat two ticks after moving to the tile.

<div style="text-align:center"><img src="https://i.imgur.com/56GrmhD.gif" alt='Xarpus poisoned tile' width=500>

On tick 1, we move. On tick 2, our area queue goes off. On tick 3, the hitsplat is applied from our queue. These commands are spread over three ticks because both the area queue is before movement during our turn and the queue is before the area queue during our turn.

#### Interactions with objects/items and npcs/players

Interactions are one of the most useful categories for skilling. They're separated into two types: interactions with object or items and interactions with npcs or players. In between the two, movement occurs, which has many noticeable consequences in game. One of them is that interaction with a banker (an npc) happens sooner than interaction with a bank booth (an object) when moving up to them, as shown in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/5UBIyFI.gif" alt='Interactions with bank booth/banker' width=500>

In the first case, we move then interact with the banker in the same tick; in the second case, we move on a tick then interact with the bank booth on the following tick. The same behavior can be seen when moving up to a tree (an object) to woodcut or when moving up to a fishing spot (an npc) to fish.

An interesting use of client input and interactions with items is to 1-tick bones to bananas or peaches. If this method could be sustained, it would provide 150k magic experience per hour at just level 15 magic. A clip is below.

<div style="text-align:center"><img src="https://i.imgur.com/dMIadS3.gif" alt='1t bones to bananas or peaches' width=500>

Each tick, we first cast bones to peaches in client input then later pick up a bone during interactions with objects/items during our turn.

Interactions can also be cleverly used to produce an optimal method to cook. It's well known that karambwans can be cooked once per tick; however, it's less widely known that the property of karambwans that makes 1t cooking possible is that raw karambwans can produce more than one cooked product. Since raw beef and raw bear meat share this property, they can also be used to 1t cook. This holds even in f2p where no second option explicitly appears, as shown in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/5JiU7Mp.gif" alt='1t cooking' width=500>

On the first tick, our use of the raw beef on the fire is processed in client input and then a choice interface opens during interaction with objects. On the next tick, both the cook happens (finishing processing of that beef) and then the next raw beef being used on the fire is processed in client input, then an interface opens in interaction with objects. This then repeats.

## Tick Manipulation II: The Skilling Tick

Most tick manipulation in skilling amounts to carefully controlling the so-called skilling tick. There's two relevant variables for this: One is a `global tick counter` that starts at 0 and increments by one every tick before any player's client input and another is a player-specific variable called the `skilling tick` that specifies a value of the global tick counter where a skilling action can occur. 

Sometimes people refer to the skilling tick minus the global tick counter as the _skilling timer_, which can be conceptually convenient since it counts down by one every tick and the skilling action occurs when the skilling timer "goes off" and hits 0. We will refer to the skilling timer most of the times that we mention setting the value of the skilling tick: for convenience, we sometimes write that we are delaying the skilling timer by _k+1_ in place of writing that we are setting the skilling timer to _k_. This is helpful because there are _k+1_ ticks between when the timer is set and when it goes off.

Skills that use the skilling tick are firemaking, fishing, mining, woodcutting, the combat skills, and, to a lesser extent, thieving. A first example of the use of the skilling tick can be found in woodcutting, which standardly is a 4 tick action. Let's set the stage: suppose that the global tick counter is 998 and the skilling tick is 900. This means in part that it's been 98 ticks since we've completed our last skilling action. On global tick 998, let's say we click a reachable tree while there's one tile between our character and the tree. On global tick 999, our click will be processed in client input where our `interacting_with` variable will be set to the tree; later in the tick, during our turn, we will move one tile to be adjacent to the tree. On global tick 1000, during our turn, we will interact with the tree, and, since our skilling tick is less than the global tick counter, our skilling tick will be set to 1003. On global ticks 1001 and 1002, we will interact with the tree during our turn largely with no effect. On global tick 1003, which is the value of the skilling tick, our interaction with the tree during our turn will produce a roll. If the roll succeeds, we get a log and there's a possibility of the tree falling.

### Early examples

While mining, we roll for an ore against any rock that we're interacting with during the skilling tick. An immediate consequence of this is that stopping interacting with a rock then restarting interacting with a rock has the same effect as continually interacting with a rock. This can be cleverly used while mining iron in the mining guild to buy an extra tick for reaction time, as shown below.

<div style="text-align:center"><img src="https://i.imgur.com/AxUdwnW.gif" alt="Bea's mining guild iron" width=500>

Using a dragon pickaxe and crystal pickaxe provides a chance of delaying the skilling timer by two ticks rather than always three ticks. We control for this by always moving to another rock after two ticks: when the pickaxe 2 ticks, we move to an undepleted rock to set the skilling tick again; when the pickaxe 3 ticks, we deplete the rock we just moved to, and then move back to the old rock to set the skilling tick again. We also drop ore in client input before interacting with a rock to keep space in our inventory.

The skilling tick is just a number, so it doesn't know what kind of action set the skilling tick. In the iron mining example, the skilling tick both got set from an iron rock and was used to mine an iron rock; however, this consistency was not needed. We can also set the skilling tick using one action and then complete a distinct action during the skilling tick. This strategy is behind almost every optimal skilling method in the game. A first example of this is from setting the skilling timer using woodcutting (which is 4 ticks) then completing a barbarian fishing action (which is by default 5 ticks), as shown in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/FL2K5q7.gif" alt='Tree fishing' width=500>

In doing the method, we click on the tree on the same tick as the roll from fishing; this makes us interact with the tree on the next tick when the skilling timer gets delayed, which gives a shorter delay than from barbarian fishing. This method, known as _tree fishing_, was optimal near the start of Old School, before more ways to set the skilling tick were known.

### Manipulating the skilling tick

A major breakthrough in skilling occurred when the community found inventory items which could delay the skilling timer without consuming any items. This was done around the time of the Skilling Cups with the discovery that making herb tar, which takes three ticks, uses the skilling tick. With this, nearly all of fishing, mining, and woodcutting actions can now be done every three ticks. An example with barbarian fishing is below, where we use a knife on a teak log rather than an herb on swamp tar. (These two choices, knife logging and making herb tar, are essentially equivalent from the perspective of the skilling tick.)

<div style="text-align:center"><img src="https://i.imgur.com/pAPlYsy.gif" alt='Barbarian fishing' width=500>

We are knife-logging to set the skilling timer forward, then getting a barbarian fishing roll on the skilling tick. Notice that we click to use the knife on the log on the same tick as we get a fish roll or experience drop, so that the knife-log interrupts our fishing and sets the skilling timer in place of the skilling timer being set through fishing. We also could have cut a fish with our knife sometime after the previous fishing roll and before beginning again to interact with the fishing spot.


A knife-log, for example, will set the skilling timer after the skilling timer has gone off. Most actions which can change the skilling tick similarly have this _after the skilling timer has gone off_ condition. One way to make sense of this is through an example: while interacting with a fishing spot every tick, the skilling tick change condition is passed every tick, so to make getting rolls possible, the skilling tick change must only happen _after_ the roll, which occurs on the skilling tick. Combat and firemaking don't follow this, though: they set the skilling tick on the action, emulating the effect of a "cooldown".

#### Eating to delay the skilling tick

Not all skilling tick changes have this _after the skilling timer has gone off_ condition, though. Eating a non-karambwan food item will add three ticks to the skilling tick. For example, if the skilling tick is in the past, eating a non-karambwan food won't necessarily bring the skilling tick into the future; however, when the skilling timer is -1, eating (for example) a roe or caviar moves the skilling timer to 2, which is the same as effect as from a knife-log. Sometimes this is referred to as eating "continuing cycles". An example is shown below, where we optionally also cut a fish after delaying the skilling timer by eating.

<div style="text-align:center"><img src="https://i.imgur.com/evNmIjA.gif" alt='Cut-eat 3t barb fishing' width=500>

Karambwans add two ticks to the skilling tick, rather than three like other foods. Therefore karambwans can be used 2.5t, such as at barbarian fishing in the clip below. 2.5t fishing means that we alternative between getting a roll in two ticks and three ticks.

<div style="text-align:center"><img src="https://i.imgur.com/hAIRd8S.gif" alt='2.5t barb fishing' width=500>

#### Flinching

Being attacked can effect the skilling tick. In more detail, while our auto-retaliate is on, we are not interacting with anything, and our skilling tick is in the past, being attacked will set our skilling tick to half of our attack speed, rounded down. This is called being _flinched_. This mechanic was likely designed for use in combat, but it is used in several optimal skilling methods.

With a 2 tick or 3 tick weapon, being flinched will make the skilling tick be the next tick. While being attacked every other tick, this can make the skilling tick be set to be the next tick every other tick, as shown in the example below.

<div style="text-align:center"><img src="https://i.imgur.com/BQR6DDl.gif" alt='2t prif teaks' width=500>

This method is 2t teaks, and was the optimal method for woodcutting experience and the pet before Fossil Island introduced a spot which allows for 1.5t teaks. The method involves a two tick rhythm: on the first tick, we remove interaction with a tree in client input, then the npc attacks during its turn, and then we receive the hitsplat during our turn, which has us set the skilling tick and start interacting with the attacking rabbit. On the second tick, we set our interaction to a tree in client input and then interact with the tree during our turn to get a roll. The interaction with the tree at the beginning of the first tick can be removed in a number of ways, including via a no-movement path (as in the clip), log dropping, or dart fletching. Notice that we are at no point interacting with a rabbit during any skilling tick.

Essentially the same method is also done for swordfish harpooning, as shown below.

<div style="text-align:center"><img src="https://i.imgur.com/5QyQZVc.gif" alt='2t pisc swordfish harpooning' width=500>

Interestingly, in this variant of the method, we don't need to make any clicks to cancel our interaction with the fishing spot. This is because harpoon fishing spots (like most others) force you to stop interacting with them after one tick before the skilling timer has gone off (except for when the skilling timer is 4, which allows for afk fishing). This force stop of our interaction with the fishing spot is the same reason why the rhythm for 3t swordfish fishing is different than the rhythm for 3t Barbarian fishing: we still have to start interacting with the fishing spot on the skilling tick.

#### Important special cases

Sometimes the first interaction with something can have different behavior than later interactions. For example, during barbarian fishing, the first interaction with the fishing spot will never produce a roll. This blocks the 2t methods based on flinching; however, notice that the method based on karambwan eating wasn't blocked since the eat happens in client input, allowing for one interaction with the fishing spot in the same tick. Other places where this behavior is observed is at fly fishing spots, pike fishing, and the boulder in volcanic mine.

Making herb tar, teak or mahogany stocks, or kebbit vambraces do delay the skilling timer by three after the skilling timer has gone off. However, they also do the same when making them on the skilling tick. This means that unlike woodcutting, fishing, or mining where interactions check if the skilling timer is _negative_ before setting the skilling tick, these actions check if the skilling timer is _nonpositive_ before setting the skilling tick. Celastrus bark has the same nonpositive behavior. Another example of unusual behavior can be seen in snow gathering: there, the skilling timer is delayed by three whenever the skilling timer is less than two.

This section has focused on the skilling tick, but there are other "named" ticks which function in much the same way. Particularly prominent examples are the alchemy tick (which several normal spellbook spells use), the eating tick (which control which ticks we can eat non-karambwan food), the karambwan eating tick, the pot sipping tick, and the bone bury tick. For an example of delaying these ticks, note that sipping a potion will block eating, karambwan eating, and potion sipping for three ticks. This is because sipping a potion delays the eating, karambwan eating, and the pot sipping tick. However, a non-karambwan food item will only the eating tick, so karambwans and potions can still be immediately consumed. The reason why karambwans can't be used to 2t at Barbarian fishing is because karambwans delay the karambwan eating timer by three ticks, despite delaying the skilling timer by two ticks.

## Tick Manipulation III: Stalls

When our character is stalled, our client input and our turn is completely blocked. Our timers don't go off, our queue doesn't evaluate, we can't interact with anything, we can't move, and no client input is accepted. Stalls appear often during skilling, sometimes to block increased experience rates and sometimes to improve experience rates.

There are of course immediate examples of unhelpful stalls. Agility obstacles stall our character while we cross them. This means that no amount of ingenuity will ever speed up crossing an obstacle, as we're blocked during crossing. Another example is picking up a box trap, which takes three ticks. Being stalled here means that taking four ticks to reset a trap, as we saw in the method above, is the fastest possible duration.

Another example where a stall slows us down but an understanding of mechanics saves us a tick compared to a naïve method is below, where our bank interface opens on the same tick that a stall ends.

<div style="text-align:center"><img src="https://i.imgur.com/us896Tz.gif" alt='Interface opening after stall' width=500>

We first process the bank click in client input, which sets the object we're interacting with to be the bank, then we process the spell click in client input, which does the spell action and produces a stall for three ticks. Note, the spell crucially does not remove our interaction. Three ticks later, the stall ends at the beginning of our turn, then the bank interface opens during 'interact with object/items'. This saves a tick over casting the spell then clicking the bank on the tick the stall ends; clicking the bank a tick before the stall ends will not be processed since the stall ends after client input.

### Stalls after movement

Some objects produce a one tick stall before fully executing our interaction with them after moving in the previous tick. For example, changing levels via most ladders in game is slowed down by one tick due to this stall. There are primarily two places where this stall appears in skilling: while woodcutting player-farmed trees or mining rocks. For example, when afk mining, this stall slows down experience when moving in between rocks: it forces us to take five ticks until a roll instead of the usual four ticks (one for movement, three for the skilling timer delay from mining).

This stall can be used to help us by giving up to two rolls on a skilling tick, rather than just one roll. Below we show a clip of 1.5t teaks on fossil island, which makes use of this stall.

<div style="text-align:center"><img src="https://i.imgur.com/LoHe2Ae.gif" alt='fossil island 1.5t teaks' width=500>

To perform the method, herb tar to set the skilling tick in client input and move, then, on the next tick, interact with a tree. That interaction will produce a stall that ends in one tick. On the third tick, the stall ends and we interact with the tree for a roll. Significantly, our interaction with the tree was paused when the stall began, so our interaction completes when the stall ends. This finishing of our interaction from the second tick is the source of the additional roll. 

Unfortunately, this method can easily go wrong. If we interact with a tree two ticks after moving rather than one, the stall won't happen and we will lose out on one roll. Further, if we move on the tick after setting the skilling tick with (say) herb tar, we will be stalled when we would have interacted with the tree during the skilling tick, giving no rolls at all.

The same method is commonly done at sandstone and granite in the quarry. There we still can get two rolls, but the second roll only offers us a second chance for when the first roll fails. This is perhaps the only example in game of tick manipulation being used to increase the probability of successfully gathering a resource in a tick.

<div style="text-align:center"><img src="https://i.imgur.com/qTXZQgk.gif" alt='3 tick 4 granite' width=500>

### Stalls on successful rolls

Most mining rocks produce a one tick stall after successfully rolling a resource. The effects of this are most well known for gem mining, where herb tar is used to produce a resource every four ticks instead of the usual three. Iron, granite, and sandstone do not have this stall. The motherlode mine only has a one tick stall for the first resource gathered per click to interact.

### Stalls in cooking

Another example of an unhelpful stall can be found when cooking food besides raw beef, raw bear meat, and raw karambwans. The 1t cooking method doesn't work for other food items due to a stall which forces the first cook of these raw food items to take four ticks. However, since the stall only appears when there's more than one of the same raw food in our inventory, it's still possible to accelerate cooking these other food items by directly circumventing this problem. Indeed, we can 2t cook by only cooking with one raw food in our inventory at a time, which we show in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/3netQ5Y.gif" alt='2t cooking' width=500>


On the first tick, we withdraw a raw shark from our bank and process our click on the stove in client input, then on interaction with object during our turn, we cook the shark. We repeat on the following ticks, every so often also depositing our inventory during client input before withdrawing a raw shark.

## Tick Manipulation IV: The Queue

Weak commands in the queue are used commonly in skilling. Skilling actions not based on interactions tend happen in the queue. For example, making herb tar uses a weak command in our queue: in client input on the tick after we click a low leveled herb onto swamp tar, the skilling tick will be set if the skilling timer is nonpositive, then a command to complete the make on the skilling tick will be placed as a weak command in the queue. Recall that we can quickly test making herb tar is indeed a weak command since rearranging our inventory cancels the action.

The queue is also used for repetitive actions, such as in fletching, potion making, cooking, wintertodt brazier feeding, smithing, and crafting. For most of these, the first make occurs in client input if the first make happens on the tick after the click, then later makes occur from the queue. In all of these examples, the command are weak. In wintertodt, the commands being weak is the reason why damage frustratingly interrupts our actions.

## Tick Manipulation V: Movement and Pathing

There are two major aspects of movement that need to be understood for skilling. The first involves being able to keep track of the position of npcs and players across ticks, and the second involves controlling which tiles are obstructed or not. 

One example is moving the rabbits between trees at prifddinas teaks without disturbing their attack schedule, as shown below.

<div style="text-align:center"><img src="https://i.imgur.com/xbDfTia.gif" alt='Movement from the third tree at prif teaks' width=500>

We first drag the rabbit with lower ID (so whose turn is first) over to the desired tile, then unobstruct the tile by running over it so that the second rabbit can also reach the tile.

Another example is the rat setup for 2t swordfish. Below is an optimal strategy for this.

<div style="text-align:center"><img src="https://i.imgur.com/G3BlmCc.gif" alt='Rat setup at 2t swordfish' width=500>

We first gets the rats horizontal by running away from the coast then back, careful to walk to the side of the stack on the way back to keep the tile next to us obstructed. Second, we move to the side of the rat line up on the tick after being attacked. This makes the second (farther south) rat attack us two ticks after the first. Third and finally, we take a diagonal step toward the coast to leave undisturbed the rat set up. The diagonal step keeps them undisturbed since they both move to the only tile adjacent to our character that is one tile away from their former position.