# Introduction #

There are many who would argue that eliminating switch bounce is important when interfacing with a microcontroller; others believe that software is (almost) free, so let the CPU handle it.

I belong to the second group.  If you are creating a product and have to distribute 1000 units, adding a couple of capacitors to each board could set you back `$0.20 * 1000 == $200`.  Wouldn't you rather have that money in your pocket?

# Impact of Bounce on the MCU #
The AdaEncoder code is designed with the following in mind:
  1. Switches will be bouncy
  1. A human will be turning the switch
  1. A couple of missed detent positions never hurt anybody

The question is, how bouncy is bouncy?  How much of an impact does that have on your code?

Given a fast enough bounce, the ATmega chip in your Arduino may not have time enough to recognize it as a state change, and an interrupt will not take place.  Therefore,
  * We're not too worried about really fast bounces as the chip couldn't recognize them anyway.
How fast is really fast?  I don't know.  Suffice it to say that in my experimentation with this code, the number of state changes between the detents is not inordinately large.  During my rotations, I would not see a dozen state changes where I ought to see only four (If we write down the pins in A,B form, the state changes are: 1,1 -> 1,0 -> 0,0 -> 0,1 -> 1,1).  Generally I might see 4, or 5, or 8, or 6... but not a dozen.  Nothing that would alarm me.  The code is sufficiently fast in my opinion that a hardware debounce circuit is not necessary.

Furthermore, it is assumed that the intended purpose of the rotary encoders attached to the Arduino are as controls manipulated by a human being.  True, should this code prove sufficiently fast enough there is no reason why a motor couldn't control a rotary encoder that was read by the AdaEncoder library.  It is up to the implementor to decide if this solution is appropriate (see the Speed Wiki page).  But for the cheaper (which is approximately the same as "bouncier") switches, with limited lifespans and of questionable reliability (at least for such an application), it seems to me that the assumption that this code is to be used on encoders that are rotated by humans is sound.

Thus the conclusion:  Don't worry about bounce.  The ATmega328 has enough oomph and the interrupt code has enough elegance to handle it.  Paying a few cents for some debounce capacitors is up to you, but I don't recommend it.