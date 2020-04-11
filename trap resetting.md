# Trap Resetting

This purpose of this guide is to explain optimal trap resetting in a way that makes clear the underlying mechanics. The method explained here can be used for any kind of chinchompas, but we will set up five non-adjacent traps as if we were hunting red chinchompas.

Trap resetting in chinchompa hunting involves two steps: laying traps and picking up traps. By default, picking up a box trap takes three ticks and laying a box trap takes four ticks. Our character is stalled while we pick up traps, so there's no possible way that we can shorten the pick up duration. However, traps lay on the skilling tick, so we can use usual tick manipulation methods to shorten the lay duration.

## 7t trap resetting with herb tar

The clip below shows such a method where we shorten the duration of a trap lay. Some call this method "3t trap resetting" or "3t hunter" because the lay takes three ticks. (Those who do are definitely nerds though.)

<div style="text-align:center"><img src="https://i.imgur.com/A1zzQJ3.gif" alt='trap resetting with herb tar' width=500>


In this method, we spend seven total ticks to reset a trap: one tick to move to the trap, three ticks to pick up the trap, and only three ticks to lay a new trap. The trap lay was one tick shorter than usual because we delayed the skilling timer by three ticks by starting to make herb tar, rather than by four tick by putting down a trap.

## 5t trap resetting with celastrus bark

A recently introduced item, celastrus bark, can be used to delay the skilling timer by four ticks. This typically would be uniformly worse than delaying by three ticks, however it does contribute to the optimal method for collecting chinchompas. The key idea is that picking up a trap fits snugly between delaying the skilling timer and laying a trap when the timer goes off four ticks later.

<div style="text-align:center"><img src="https://i.imgur.com/zugJnZi.gif" alt='trap resetting with celastrus bark' width=500>


In this method, we spend five total ticks: one tick to move to the trap, three ticks to pick up the trap, and one tick to lay a new trap. We used celastrus bark to set the skilling timer to three ticks on the same tick as we started picking up a trap. (Notice that we clicked the bark on the same tick as we clicked to pick up the box trap on the ground.) When the skilling timer hits zero, four ticks after we click on the bark, a box trap instantly lays from our inventory. During the first three ticks, instead of essentially idling, we picked up the trap. 

## 4t trap resetting

The previous methods focused on resetting a single trap. Surprisingly, momentum from laying a previous trap can be used to reset a next trap in only four ticks. To do this, we must crucially make use of a mechanic that allows us to lay a trap and move in the same tick, as shown in the clip below.

<div style="text-align:center"><img src="https://i.imgur.com/C4md7tu.gif" alt='Laying a trap and moving in the same tick' width=500>


Here, when the skilling tick was on the next tick, we clicked to lay a trap from our inventory and clicked to move. Keep in mind that laying a trap delays the skilling tick by four, which we used in the clip to set the skilling timer.

Using our ability to do this, we can spend _zero_ ticks on movement after resetting the first trap. In comparison with 5t trap resetting with celastrus bark, instead of moving then delaying the skilling timer on a separate tick, we are doing them both on the same tick.

<div style="text-align:center"><img src="https://i.imgur.com/qPOZtfQ.gif" alt='Resetting all five traps' width=500>

The full method based on this lays a trap, delays the skilling timer, and moves in the same tick and repeats this every four ticks; in the method, we pick up traps in between those ticks.

When hunting black chinchompas in the wilderness, the same method works. However, we can instead lay a trap and start picking up an adjacent trap on the same tick, and then move immediately after picking up the trap.

## Trap resetting while hunting

While trap resetting in practice, we don't stick to just one of the above methods. We actually use all three of them! Whenever possible, we 4t trap reset. This means that if there's other traps that need resetting while we are resetting one trap, we immediately reset those too. To reset the first trap, we prefer to use 5t trap resetting. However, when the skilling timer is positive, celastrus bark cannot delay the skilling timer. This makes us have to first pick up the trap to allow our timer to become nonpositive, forcing us to use 7t trap resetting. Put differently, we must use 7t trap resetting over 5t trap resetting when there's fewer than three ticks in between finishing the last reset and starting the next.
