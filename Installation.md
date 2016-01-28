# Introduction #

Installing this library requires that you know where the Arduino libraries folder is.  On the Mac, by default it's is `~/Documents/Arduino/libraries/`.  On the PC it will be in someplace similar, I believe it is a subfolder of the My Documents folder; probably there is an Arduino folder directly under it.  In any event, you can check the Preferences menu in the Arduino IDE and it will show you where your sketchbook location is.

Inside that location, you must have a libraries directory (aka 'folder').  If it's not there, create it.

# Details #

See http://www.arduino.cc/en/Reference/Libraries.

  1. Download the `adaencoder.zip` file found in the Downloads section.  See the Downloads link, above.  Or click here: http://code.google.com/p/adaencoder/downloads/list
  1. On the Mac, assuming your web browser puts the file in a Downloads directory under your home directory:
```
cd ~/Documents/Arduino/libraries; unzip ~/Downloads/adaencoder.zip
```
(change that cd command to be where your sketchbook directory is)
  1. On the PC, go into the zip file and drag the folders `AdaEncoder` into the sketchbook location.
  1. Grab the latest Pin Change Interrupt library for your AdaEncoder:
    * For version 0.7beta or newer, go to http://code.google.com/p/oopinchangeint/ and get the latest ooPinChangeInt library. Installation instructions are in the wiki at http://code.google.com/p/oopinchangeint/wiki/Installation
    * For earlier versions, go to http://code.google.com/p/arduino-pinchangeint/downloads/list and get the latest PinChangeInt library; you must use version 1.70beta or newer.  Installation instructions are at http://code.google.com/p/arduino-pinchangeint/wiki/Installation
  1. restart your Arduino IDE
  1. See the Usage Wiki page here for information on how to use the library.