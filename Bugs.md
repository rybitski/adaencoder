# Introduction #

There are indeed bugs in this code.  More details to come.  There is one known bug in the code that I can think of.  :-(


# Details #
## [https://code.google.com/p/adaencoder/issues/detail?id=](https://code.google.com/p/adaencoder/issues/detail?id=): genie() Does not Loop Completely ##
When we loop over a set of more than 1 encoder, we may not loop properly through all of them.  Because the code works like this:

  * Loop over the encoders.
  * If you find an encoder with a non-zero number of registered clicks, return it.

The problem is, what if you get to the last encoder?  It looks for the next encoder, discovers that it is NULL, and returns.  However, at that point the first encoder may have registered a turn or two.

The fix is to remember the encoder at which you are starting, then loop through the encoders.  If you get to NULL, go to the first encoder.  As soon as you reach the encoder that you started at, you're done.

Again, I am reluctant to fix this bug as:
  1. How many multiple-encoder installations are y'all likely to create?  And,
  1. Of those, how many are NOT going to be in an Arduino sketch loop() that is not constantly checking the encoders to see which one has been turned recently?  I'd wager, very few.

So:  This will not be fixed unless I receive a request.  Otherwise, Caveat Programmer.