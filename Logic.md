# Introduction #

There are two problems to solve when interpreting output from a rotary encoder:
  1. What do the pin values signify?
  1. What to do about switch bounce?

# Pin Values #
## or:  "Will that be CW or CCW, sir?" ##
## Encoder Operation ##
There are many sources that describe the operation of rotary encoders on the Internet.  A simple Google search will find them. In the case of these two inexpensive encoders (see http://www.adafruit.com/products/377 or http://www.sparkfun.com/products/9117), we have a device with a common center pin and two pins whose state you read, which I'll call Pin A and Pin B.  As you turn the encoder's shaft, the switches open and close in a particular pattern.

Assume that the common pin is connected to ground, and the two pins are connected to digital pins on the Arduino.  This is the easiest connection method because it requires no additional components with the Arduino.  You can turn on the Arduino's internal resistors so that a reading on a pin becomes a "1" when the switch is open.  With the switch closed, the common is connected to the Pin and a 0 is read.

The switch starts out in the detent position with both switches open.  So you will read a 1 on Pin A and a 1 on Pin B.  As you rotate it, one of the switches closes.  So now you read 0,1 or 1,0 on Pins A and B, depending on the direction you've turned.  Continue turning, and both switches will close together; you will read 0,0 on the two pins.  Further, the next reading will be 1,0 or 0,1- the opposite pin output from the beginning.  Finally, you hit the detent position and you read 1,1 again.

So you can see that you can tell which direction you went, but there's no absolute reading.  You can't tell if the switch is now 30 degrees from the beginning, or 60 degrees, or what have you.  You only know you clicked 1 detent in 1 direction, either clockwise (CW) or counterclockwise (CCW).

## Software Logic ##
There are many examples of code that follow the state changes of Pins A and B, in order to show you the complete operation of the encoder.  In this code, I am interested only in the detent positions: When have I advanced a "click".  This is because I am interested in a human turning the encoder.  A human user is likely going to count "clicks".  They are unaware that there are 4 states between clicks, so I am uninterested in those states- except that they tell me which way the encoder is going.

Thus, I am able to cheat a bit.

Note that the state transitions for the encoder are going to go like this:
```
1,1 -> 0,1 -> 0,0 -> 1,0 -> 1,1
```
or
```
1,1 -> 1,0 -> 0,0 -> 0,1 -> 1,1
```
Note well, too, that that's not entirely true.  See "Switch Bounce", below.

Well, given that those are the states of a theoretical encoder, we note:  If you transition from `1,1 -> 0,1` you are going in a direction.  If you transition from `1,1 -> 1,0` you are going in the other.

How cool is that?!  Not very cool, you think, because you need to get to 1,1 again to mark the detent-to-detent turn as complete.  Not only that, but what if the user just nudges the switch, and falls back to the original detent?  Or- more likely- what if the switch bounces?  That is, it transitions from `1,1 -> 0,1 -> 1,1 -> 0,1`?

Notice this:  between every transition from detent-to-detent, you must pass through the 0,0 state.

With that statement, we have everything we need to mark a move from one detent to the next:
  1. We pass from 1,1 to 0,1 or 1,0.  This tells us the direction, but we don't know if we're going to go through with it.
  1. We pass from there to 0,0.  It is here that we mark the direction as "known".
  1. We pass from there to a "don't care" state, because at this point we already know which direction we're going in.
  1. We arrive back at 1,1.  Voila!  We've moved.

That is how the software logic works.  See lines 270-287 of AdaEncoder.cpp.

# Switch Bounce #
## ...or, "I'm an Encoder, and Encoders Bounce!" ##

The transitions, e.g.
```
1,1 -> 1,0 -> 0,0 -> 0,1 -> 1,1
```
are nice in theory, but in the real world, switches bounce.  See http://www.micahcarrick.com/avr-tutorial-switch-debounce.html for a nice image.

This means that to the Arduino, the transitions might actually look like this:
```
1,1 -> 1,0 -> 1,1 -> 1,0 -> 1,1 -> 1,0 -> 0,0 -> 1,0 -> 0,0 -> 0,1 -> 0,0 -> 0,1 -> 1,1
```
Yeesh.  That's ugly.

Now I'm one of those people that has come to realize the following:
```
You can reproduce software for free.  A new component always costs money.
```
So, if I can reasonably debounce my switch in software, that's my first inclination.  It so happens that the logic expressed above handles switch debounce nicely.  Let's look at the code I wrote to handle the encoder transitions:
```
270     portState=*tmpencoder->port;
271     stateA=portState & tmpencoder->bitA;
272     stateB=portState & tmpencoder->bitB;
273     if (stateA && stateB ) {                                // the detent. If we're turning, mark it.
274         if (tmpencoder->turning) {
275             tmpencoder->clicks+=tmpencoder->direction;
276         }
277         tmpencoder->turning=0; tmpencoder->direction=0;     // reset counters.
278         return;
279     }
280     if (stateA == 0 && stateB == 0) {                       // The 1/2way point.  Flag that we've reached it.
281         tmpencoder->turning=1;
282         return;
283     }
284     if (tmpencoder->turning == 0) { // We are just starting to turn, so this will indicate direction
285         if (stateA) { tmpencoder->direction=1; return;  }; // CCW.
286         if (stateB) { tmpencoder->direction=-1; return; }; // CW.
287     }
```
Let's review my logic that I stated earlier:
  1. We pass from 1,1 to 0,1 or 1,0.  This tells us the direction, but we don't know if we're going to go through with it.
  1. We pass from there to 0,0.  It is here that we mark the direction as "known".
  1. We pass from there to a "don't care" state, because at this point we already know which direction we're going in.
  1. We arrive back at 1,1.  Voila!  We've moved.
Here is how the code realizes that logic:

First, lines 270-272 are not part of the logic.  They read the pins for us.  Now:
  1. We pass from 1,1 to 0,1 or 1,0:  Lines 284-287.  Here, I've utilized the fact that I've already checked for the 0,0 or 1,1 states.  So either Pin A is 1 or Pin B is 1.  If that is the case, the other one must be 0.  So, as long as I have not marked this encoder as in the "turning" state, I can mark down the direction.
  1. We pass from there to 0,0: Lines 280-283.  Easy peasy; if I reach this point, I mark the "turning" flag.
  1. We pass from there to a "don't care" state:  This is handled intrinsically by the code.  The sequence of `if` statements check for:  The 1,1 state, the 0,0 state, and then the "are we turning?" state.  If we're turning, we fall through and ignore this interrupt.
  1. We arrive back at 1,1: lines 273-279.  If we're turning, add the direction (either +1 or -1) to the "clicks" variable.
And there, lurking in lines 277, lies magic.  Because no matter what, we reset our "turning" and "direction" variables.  This is critical:  Once we hit the detent position, we're done.  We've turned... OR!  we've bounced!

Quite nicely, if we advance to 1,0 or 0,1 and have _not_ flagged the `turning` variable, but **have** flagged the `direction` variable, it doesn't matter.  Because we haven't passed through to the 1,1 state, we must be in a bounce situation.  So reset `direction` back to 0.  We're starting all over again.

This is safe because if we _are_ bouncing, we'll simply pass back into the 1,0 or 0,1 state again.  Our `direction` variable will be set, again, and we'll move on.

Getting to the 1,1 state is again interesting.  When we reach it, `turning` is set.  At this point, too, we've already set `direction`.  Now let's say we've transitioned from 1,0 -> 1,1.  If we bounce back to 1,0, we are going to fall through the `if` statement at line 273, we'll fall through the `if` statement at line 280, we'll fall through the `if` statement at line 284: in other words, nothing happens.  The `turning` variable is set on _the very first_ touch of 1,1, so any bounces back to 1,0 or movement forward to the opposite state, 0,1, are ignored.

Then, the first touch to state 0,0 means we've completed our turn.  The `clicks` variable is updated, and the switch can bounce along after that all it wants.  The only thing that happens during those bounces is that `direction` is set, but they are meaningless because it's the last `direction` set that sticks.  This would be the final `direction` setting prior to entering another 1,1 state.

And so on.  Most of the interrupt code is comprised of checking 1, 2, or 3 states with `if` statements.  It's quite lean and you bail out of there soon enough.