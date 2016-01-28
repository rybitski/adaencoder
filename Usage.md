# Introduction #

This library was intended to make it easy for you to use 1 or more rotary encoders as found at Sparkfun or Adafruit on your Arduino.  Don't worry about debouncing, the library's functions take care of that for you.

The library uses tight code (no `digitalRead()` or `digitalWrite()`) and in use requires only two methods.  The amount of code in the interrupt routines is kept small.  The library is interrupt-driven, so you don't have to intersperse time-critical polling routines within your sketch (although ultimately you need to poll some variables to determine if the encoders have been manipulated).

# Overview #
  * Connect the controller to 2 Arduino pins.  These pins must be on the same ATmega CPU PORT.  The Arduino's pins are shown below, in parentheses.  The corresponding pin numbers on the ATmega 328p are shown alongside the diagram of the 28-pin dip package:
```
                  +-\/-+
            PC6  1|    |28  PC5 (AI 5)
      (D 0) PD0  2|    |27  PC4 (AI 4)
      (D 1) PD1  3|    |26  PC3 (AI 3)
      (D 2) PD2  4|    |25  PC2 (AI 2)
 PWM+ (D 3) PD3  5|    |24  PC1 (AI 1)
      (D 4) PD4  6|    |23  PC0 (AI 0)
            VCC  7|    |22  GND
            GND  8|    |21  AREF
            PB6  9|    |20  AVCC
            PB7 10|    |19  PB5 (D 13)
 PWM+ (D 5) PD5 11|    |18  PB4 (D 12)
 PWM+ (D 6) PD6 12|    |17  PB3 (D 11) PWM
      (D 7) PD7 13|    |16  PB2 (D 10) PWM
      (D 8) PB0 14|    |15  PB1 (D 9) PWM
                  +----+
```
The ATmega 328p's pins are grouped in sets of “Port”s, shown as “PDx”, “PBx”, “PCx” above, where x represents an integer from 0 to 8.  So you can see that Port B, pin 0 is pin 14 on the chip, or D8 on the Arduino board.  You must use two pins from the same Port; e.g., D 5 and D 6, for each encoder.  You cannot, for example, use D 13 and AI 0 (aka, Analog Input 0).  D 13 is on Port B (pin 5), and AI 0 is on Port C (pin 1).

The lower-numbered Arduino pin will be known as Pin A, the higher-numbered pin as Pin B.  Here's a rough schematic, in ASCII:
```
               +-- A --> Arduino pinA
  +-- common --+
  |            +-- B --> Arduino pinB
 gnd
```
  * Include the two .h files in your .pde:
```
#include <PinChangeInt.h> // necessary otherwise we get undefined reference errors.
#include <AdaEncoder.h>
```
  * In `setup()`, run the `adaEncoder()` method; e.g.:
```
AdaEncoder::addEncoder('a', a_PINA, a_PINB);
```
  * In `loop()`, check your encoder's clicks count on a regular basis, using the `AdaEncoder::genie(int8_t *clicks, char *id)` method, or the `AdaEncoder::genie()` method.  They both return a pointer to an encoder.  `encoder->clicks` will show you if the encoder has been turned clockwise or counterclockwise, and how many times it has been turned since the last update.  Or, you can give `genie()` a pointer to a variable that will get filled with the number of clicks clockwise or counterclockwise.  See the examples below.
  * Note:  It is possible for the encoder to be turned in one direction and then turned backwards.  The library does not keep track of the encoder's state changes; it only reflects the sum total of the rotations since the last time the encoder was checked.
  * If you have more than one encoder, they will be added to a linked list that you can traverse in `loop()` to look for changed pins.  See the examples in the code below.

# Examples #
Here is an example utilization, for 2 encoders.

## Example 1, Easy Method ##
In this example, we use the genie() method with 2 arguments.  Genie performs the following tasks for you:
  * Returns the next encoder that has been rotated since the last time you checked.  You can use this to query the internal members of the encoder structure.
  * If you give it an int8\_t variable pointer as the first argument, it will insert the number of detent positions that have accumulated.  Negative numbers mean the encoder was rotated in a net counter-clockwise direction.  Positive numbers mean the encoder was rotated in a net clockwise direction.  It will adjust its internal movement counter plus or minus 1, whichever is necessary to bring its absolute value closer to 0.
  * If you give it a char variable pointer as its second argument, it will return the id of the encoder in it.
```
#include <PinChangeInt.h> // necessary otherwise we get undefined reference errors.
#include <AdaEncoder.h>

#define a_PINA 2
#define a_PINB 3
#define b_PINA 5
#define b_PINB 6

int8_t clicks=0;
char id=0;

void setup()
{
  Serial.begin(115200);
  AdaEncoder::addEncoder('a', a_PINA, a_PINB);
  AdaEncoder::addEncoder('b', b_PINA, b_PINB);  
}

void loop()
{
  encoder *thisEncoder;
  thisEncoder=AdaEncoder::genie(&clicks, &id);
  if (thisEncoder != NULL) {
    Serial.print(id); Serial.print(':');
    if (clicks > 0) {
      Serial.println(" CW");
    }
    if (clicks < 0) {
       Serial.println(" CCW");
    }
  }
}
```

## Example 2, the 'You Like to Sweat' Method ##
You are a glutton for punishment or a control freak, you just don't trust me, or you have a damn good reason for doing so.  Whatever the case may be, you want to manage the internals of the encoder structures yourself.

Well go for it.  See below for an example about how to do so.  The only gotcha to remember:
  * The `genie()` method doesn't adjust your number of clicks upon reading it.  So the encoder is endlessly updated with the net sum of encoder clicks, positive or negative.  Deal with it as you will.  In the below example, I essentially duplicate the behavior of the above sketch:  After every read of a turned encoder, I adjust the detent counter (the 'clicks' variable) 1 count closer to 0.

```
#include <PinChangeInt.h> // necessary otherwise we get undefined reference errors.
#include <AdaEncoder.h>

#define a_PINA 2
#define a_PINB 3
#define b_PINA 5
#define b_PINB 6

void setup()
{
  Serial.begin(115200);
  AdaEncoder::addEncoder('a', a_PINA, a_PINB);
  AdaEncoder::addEncoder('b', b_PINA, b_PINB);  
}

void loop()
{
  encoder *thisEncoder;
  thisEncoder=AdaEncoder::genie();
  if (thisEncoder != NULL) {
    Serial.print(thisEncoder->id); Serial.print(':');
    if (thisEncoder->clicks > 0) {
      Serial.println(" CW"); thisEncoder->clicks--;
    }
    if (thisEncoder->clicks < 0) {
       Serial.println(" CCW"); thisEncoder->clicks++;
    }
  }
}
```