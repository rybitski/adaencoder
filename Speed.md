# Introduction #

When running an interrupt, the key is to get in and out as quickly as possible.  So how fast is this code?

We review this test code.  NOTE:  This uses software interrupts, so I actually turned the arduino pins into OUTPUTs!  This means I had to comment out the following lines from the AdaEncoder.cpp file:
```
    //pinMode(pinA, INPUT); pinMode(pinB, INPUT);  // MIKE DEBUG
    //digitalWrite(pinA, HIGH); digitalWrite(pinB, HIGH);
```
This code will NOT work with the AdaEncoder.cpp file, as-is.  You must make that change.  DON'T forget to put it back when you're done playing!  (If you so choose)

```
#include "pins_arduino.h"
#include <PinChangeInt.h> // necessary otherwise we get undefined reference errors.
#include <AdaEncoder.h>

volatile uint8_t *output_port;
uint8_t output_mask;
uint8_t not_output_mask;

// PINA
volatile uint8_t *pinA_output_port;
volatile uint8_t *pinA_input_port;
uint8_t pinA_mask;
uint8_t not_pinA_mask;
// PINB
volatile uint8_t *pinB_output_port;
volatile uint8_t *pinB_input_port;
uint8_t pinB_mask;
uint8_t not_pinB_mask;

#define PINA 8
#define PINB 9
#define PINLED 13

int8_t clicks=0;
char id=0;

void show() {
  uint8_t statusA=*pinA_output_port & pinA_mask;
  uint8_t statusB=*pinB_output_port & pinB_mask;
  Serial.print("Status A: "); Serial.print(statusA, BIN);
  Serial.print(" Status B: "); Serial.println(statusB, BIN);
}

void setup() {
  encoder *myEncoder;
  Serial.begin(115200);
  pinMode(PINLED, OUTPUT); digitalWrite(PINLED, LOW);
  pinMode(PINA, OUTPUT); digitalWrite(PINA, HIGH);
  pinMode(PINB, OUTPUT); digitalWrite(PINB, HIGH);
  
  output_port=portOutputRegister(digitalPinToPort(PINLED));
  output_mask=digitalPinToBitMask(PINLED);
  not_output_mask=output_mask^0xFF;
  // set up ports for output
  pinA_output_port=portOutputRegister(digitalPinToPort(PINA));
  pinA_input_port=portInputRegister(digitalPinToPort(PINA));
  pinA_mask=digitalPinToBitMask(PINA);
  not_pinA_mask=pinA_mask^0xFF;
  pinB_output_port=portOutputRegister(digitalPinToPort(PINB));
  pinB_input_port=portInputRegister(digitalPinToPort(PINA));
  pinB_mask=digitalPinToBitMask(PINB);
  not_pinB_mask=pinB_mask^0xFF;

  AdaEncoder::addEncoder('a', PINA, PINB);
  //*output_port|=output_mask;
  /* SOFTWARE INTERRUPTS */
  /* BUG:  FIRST TIME THROUGH, IT REPORTS THE WRONG ARDUINO PIN.
  IF YOU MODIFY pinA, it reports 9 as the pin, if you modify pinB, it reports 8.
  The situation is corrected after the first interrupt!?  WEIRD!  Must fix. */
  *pinB_output_port&=not_pinB_mask; // pinB to 0 ******************************
  *pinB_output_port|=pinB_mask; // pinB to 1 ******************************
  Serial.println("Start: ");
}

int i=0;
void loop() {
  Serial.print("Start: ");
  Serial.println(i, DEC);
  delay(2000);
  //show();
  *output_port|=output_mask;     // LED HIGH
  *pinB_output_port&=not_pinB_mask; // pinB to 0 ****************************** 15.2 us
  *output_port&=not_output_mask; // LED LOW
  //                                   0 -> 1 ~ 6-7 us
  //show();
  *output_port|=output_mask;     // LED HIGH
  *pinA_output_port&=not_pinA_mask; // pinA to 0 ****************************** 13.4 us
  *output_port&=not_output_mask; // LED LOW
  //                                   0 -> 1 ~ 6-7 us
  //show();
  *output_port|=output_mask;     // LED HIGH
  *pinB_output_port|=pinB_mask; // pinB to 1 ****************************** 14.8 us
  *output_port&=not_output_mask; // LED LOW
  //                                   0 -> 1 ~ 6-7 us  
  //show();
  *output_port|=output_mask;     // LED HIGH
  *pinA_output_port|=pinA_mask; // pinA to 1 ****************************** 15.4 us
  *output_port&=not_output_mask; // LED LOW
  i++;
}
```
The speed readings are as shown above.  Salient points:
  * It takes 6-7 microseconds for the Arduino to traverse from the LED low state to the LED high.  This is a pretty simple operation.  The interrupt code takes only about 14-15 microseconds for each routine.  Not too shabby.

In summation, the entire section of Interrupt code, above, from the first LED HIGH line to the final LED LOW takes about 83 microseconds, conservatively.  This includes an extra 7 microseconds or so to allow for the fact that the next statement in real code, after the LED LOW, will take some time to execute.

83 microseconds corresponds to a frequency of 12 kilohertz.

The Sparkfun encoder, `sku: COM-09117` is a 12 step encoder.  Therefore, it can be turned at a rate of 1000 revolutions per second (60,000 rpm) and this code would just be able to keep up.  This is assuming, however, no bounces (which is impossible without hardware debounce circuitry).

The Ada encoder, `ID: 377`, is a 24 step encoder.  Therefore, it can be turned at a rate of 500 revolutions per second (30,000 rpm).  The usual caveats apply:  bounces will mess up the rotation count.  That said, I don't think these encoders were built to attach to a motor shaft.

# Interesting Glance at the Arduino's Speed #
Just out of curiosity, how fast can the Arduino set and reset a port?

We measure the speed of the following sketch:
```
#include "pins_arduino.h"

volatile uint8_t *output_port;
uint8_t output_mask;
uint8_t not_output_mask;

void setup() {
  output_port=portOutputRegister(digitalPinToPort(13)); // if doing output, or,
  output_mask=digitalPinToBitMask(13);    // defines the actual pin on that port
  not_output_mask=output_mask^0xFF;
}

void loop() {
  *output_port|=output_mask; // sets it high
  *output_port&=not_output_mask; // sets it low
}
```
Using a digital oscilloscope, I can see that the wavelength appears as:
```
700 ns high, 1300 ns (1.3 us) low
```
...Thus, a 16 MHz ATmega 328 is able to produce a waveform of about 500kHz this way.

# Digital Storage Oscilloscope #
All right you youngsters- you've got a SWEET life ahead of you.  I'm 51 years of age, and I'm working on the Arduino with the first scope I've ever owned, a Rigol DS1052E.  ...Introductory level, but MAN, is it COOL!  I could not dream of having this kind of capability when I was your age.  Appreciate it.  And- if you like to geek around with the bits and bytes, the wires and resistors, then get a scope.  You'll be glad you did, and you'll have a hell of a fun time!

This has been an unpaid announcement from Digital Storage Oscilloscopes everywhere.  I don't care which brand you get, just get one.  It's so cool.