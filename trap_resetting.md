# Trap Resetting

This purpose of this guide is to explain optimal trap resetting in a way that makes clear each  component. The method explained here can be used for any kind of chinchompas.

Trap resetting in chinchompa hunting involves two steps: laying traps and picking up traps. By default, picking up a box trap takes three ticks and laying a box trap takes four ticks. Our character is stalled while we pick up traps, so there's no possible way that we can shorten the pick up duration. However, traps lay on the skilling tick, so we can use usual tick manipulation methods to shorten the lay duration.

## 7t trap resetting with herb tar

The clip below shows such a method where we shorten the duration of a trap lay. Some call this method "3t trap resetting" or "3t hunter" because the lay takes three ticks.

<div style="text-align:center"><img src="https://i.imgur.com/A1zzQJ3.gif" alt='trap resetting with herb tar from afar' width=500>

In this method, we spend seven total ticks to reset a trap: one tick to move to the trap, three ticks to pick up the trap, and only three ticks to lay a new trap. The trap lay was one tick shorter than usual because we delayed the skilling timer by three ticks by starting to make herb tar, rather than by four tick by putting down a trap.

## 6t trap resetting with herb tar

The first tick of movement can be removed from the 7t trap resetting method when our character is adjacent to the trap. This is because traps can be picked up while we are adjacent to them, as shown below.

<div style="text-align:center"><img src="https://i.imgur.com/xZbj6rC.gif" alt='trap resetting with herb tar while close' width=500>

## 5t trap resetting with celastrus bark

A recently introduced item, celastrus bark, can be used to delay the skilling timer by four ticks. This typically would be uniformly worse than delaying by three ticks, however it does contribute to the optimal method for collecting chinchompas. The key idea is that picking up a trap fits snugly between delaying the skilling timer and laying a trap when the timer goes off four ticks later.

<div style="text-align:center"><img src="https://i.imgur.com/SftchfH.gif" alt='trap resetting with celastrus bark' width=500>

In this method, we spend five total ticks: one tick to move to the trap, three ticks to pick up the trap, and one tick to lay a new trap. Notice that we clicked the bark on the same tick as we clicked to pick up the box trap on the ground. 

## 4t trap resetting

The previous methods focused on resetting a single trap. Surprisingly, momentum from laying a previous trap can be used to reset a next trap in only four ticks. Using our ability to lay a trap and move on the same tick, we can spend _zero_ ticks on movement after resetting a trap. In comparison with 5t trap resetting with celastrus bark, instead of moving then delaying the skilling timer on a separate tick, we are doing them both on the same tick.

<div style="text-align:center"><img src="https://i.imgur.com/xVJZOuv.gif" alt='4t resetting nonadjacent traps' width=500>

In the clip, we start off by doing 5t trap resetting. The full method based on this does all of the following on every fourth tick: lays a trap, delays the skilling timer, and moves; in the three ticks between laying the traps, we pick up traps.

When hunting black chinchompas in the wilderness, the same method works. However, we can instead lay a trap and start picking up an adjacent trap on the same tick, and then move immediately after picking up the trap, as shown below

<div style="text-align:center"><img src="https://i.imgur.com/1WD0pBv.gif" alt='4t resetting adjacent traps' width=500>


## Trap resetting while hunting

While trap resetting in practice, we don't stick to just one of the above methods. We actually use all four of them! Put briefly, we prefer the methods in the reverse order of the sections.

Whenever possible, we 4t trap reset. This means that if there's other traps that need resetting while we are resetting one trap, we immediately reset those too. To reset the first trap, we prefer to use 5t trap resetting. However, when there's fewer than three ticks in between finishing laying the last trap and starting to pick up the next trap, the delaying the skilling timer is not possible. This makes us have to first pick up the trap then later delay our skilling timer, forcing us to use 6t trap resetting or 7t trap resetting. Further, when adjacent to a trap, we use 6t trap resetting over 7t trap resetting.
