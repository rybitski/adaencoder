_What's New? Jul. 30, 2014n: **Version 0.8-rc1 has been released. Mostly this update is to open the license to Apache-2.0 from the GPL-3.0. Requires the ooPinChangeInt library, which can be found here: http://code.google.com/p/oopinchangeint/ . To Download the library, do not use the Downloads menu item, above. Click on the link in the lower left menu.**_

# Introduction #
This library interfaces with 2-pin encoders (2 pins A and B, then a common pin C).  It does not indicate every state change, rather, it reports only when the decoder is turned from one detent position to the next.  It is interrupt-driven and designed to be fast and easy to use.  The interrupt routine is lightweight, and the programmer is then able to read the direction the encoder turned at their leisure (within reason; what's important is that the library is reasonably forgiving).  The library is designed to be easy to use (it bears repeating :-) ) and it is reasonably immune to switch bounce.
# Dependencies #
The latest versions (0.7beta or above) depend on ooPinChangeInt version 1.03beta or higher, found here: http://code.google.com/p/oopinchangeint/

Version 0.5 depends on the PinChangeInt library, found here: http://code.google.com/p/arduino-pinchangeint/ . I have not tested version 0.5 to see if it works with up-to-date versions of the PinChangeInt library, although I imagine it will.

Earlier version depended on a modified version of the PinChangeInt library.  The files are included in the `adaencoder-(version).zip` file for download.

Library was tested with multiple AdaFruit and Sparkfun encoders:

https://www.adafruit.com/products/377

http://www.sparkfun.com/products/9117

# For More Information #
Click on the Wiki link, above.