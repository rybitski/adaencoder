# Introduction #

This library depends on the common of your encoders wired to ground.  The two pins you use will be set as inputs, and they will have the resistor to Vcc turned on.  Connect the two signal pins of the encoder to those Arduino input pins.

# Details #

## Connecting ##
Connect the controller to 2 Arduino pins.  These pins must be on the same ATmega CPU PORT.  See the .cpp file for more information.  The lower-numbered Arduino pin will be known as Pin A, the higher-numbered pin as Pin B.  Here's a rough schematic, in ASCII:
```
               +-- A --> Arduino pinA
  +-- common --+
  |            +-- B --> Arduino pinB
 gnd
```

## Caveats ##
Refer to the pin chart at http://www.arduino.cc/en/Hacking/PinMapping168.

The ATmega328 and its kind (ATmega168, ATmega2560) all use PORTs for their input and outputs.  A PORT is essentially a group of pins on the ATmega processor.  They are interesting because the pins grouped in each PORT share some things in common.  For example, you can read all the pins in a PORT in one command.  What this means for you as a designer is that you should NOT use two different PORTs for the two pins of your rotary encoder.

How do you know which pins are common with which PORTs?  Look at the pin mapping diagram as given in the link, above.  The pin names closest to the IC chip:  ie, PD0, PD1, PB6, etc., show you the PORTs.  B, C, and D are the three PORTs available on the ATmega168 and 328.  There are more PORTs on the ATmega2560 used in the Arduino Mega, but only those 3 PORTs allow for Pin Change interrupts on those bigger chips, too.

So, when you connect your rotary encoder to your Arduino, make sure that its two pins- which I call PinA and PinB- both attach to the same PORT.  This is a summary of the Arduino-to-port mappings that are available to you:
```
 Arduino Pins         PORT
 ------------         ----
 Digital 0-7          D
 Digital 8-13         B
 Analog  0-5          C
```

In summary, what you need to do... the ONLY thing you really need to do... is ensure that both pins of your encoder are attached to pins within those ranges, above.  Don't cross ranges and connect, for example, digital pin 7 to digital pin 10.  It won't work, and the addEncoder method will not configure your ports.