**Arduino just got motion detection - with nothing but a wire!**

**[VIDEO DEMONSTRATION](https://www.youtube.com/watch?v=4KjB-HMuUs4)**

By monitoring the amplitude of AC electricity waveforms in the air for changes, Buzz provides motion detection using only a wire! It's extremely easy to implement, and a perfect library for all experience levels.

----------
# Contents
- [Explanation](#explanation)
- [Installation](#installation)
- [Usage](#usage)
- [Functions](#functions)
- [Supported Pins](#supported-pins)
- [Limitations](#limitations)
- [Contributing](#contributing)
- [License and credits](#license-and-credits)

----------
# Explanation

Due to the ATMega328p's ADC being very high impedance, it can easily detect the AC electricity waves that leak into the air via open outlets, bad sheilding, and more.

When something statically charged (human, pet, blanket, etc.) passes near the antenna, it increases or decreases the voltage perceived at the input. Even without rubbing a balloon on your head, you'll always have enough static charge to affect this value a measurable amount.

The Buzz library allows you to easily monitor these changes, and attach your own functions that will execute when motion excedes a specified threshold.

----------
# Installation

~~**With Arduino Library Manager:**~~ Coming soon!

~~1. Open *Sketch > Include Library > Manage Libraries* in the Arduino IDE.~~

~~2. Search for "Buzz", (look for "Connor Nishijima") and select the latest version.~~

~~3. Click the Install button and Arduino will prepare the library and examples for you!~~

**Manual Install:**

1. Click "Clone or Download" above to get an "arduino-buzz-master.zip" file.
2. Extract it's contents to the libraries folder in your sketchbook.
3. Rename the folder from "arduino-buzz-master" to "Buzz".

----------
# Usage

Using the Buzz library is very simple, you only need the following to get started:

    #include "Buzz.h" // Include the Volume library
    Buzz buzz;

    void setup() {
      Serial.begin(115200);
      buzz.begin(A0,60,3000);
    }
    void loop() {
      Serial.println(buzz.level());
      delay(1);
    }

Next, connect a wire/jumper (6-12") to pin A0, and open the Arduino IDE's Serial Plotter to see the current motion value! Try waving your hand near the antenna, or walking past it.

----------
# Functions

**Buzz buzz**;

This initializes the Buzz library after import. "buzz" can be any word you want, as long as it's reflected in the rest of your code.

**buzz.begin**(byte **pin**, byte **hz**, unsigned int **coolDown**);

This sets up a Timer Compare Interrupt on Timer1 for logging motion changes. It watches the ADC input defined by **pin**, does phase cancellation for **hz** AC to remove sine-wave artifacts from the data, and waits for **coolDown** milliseconds for the ADC to stabilize before triggering any alarms.

**vol.setMasterVolume**(float **percentage**);

This is a multiplier applied to the volume of any tones played. By default this is 1.00 - a value of 0.34 would make all tones 34% of their programmed volume;

**vol.tone**(unsigned int **frequency**, byte **volume**);

*This is where the magic happens.* At the frequency you specify, your Arduino will analogWrite(**volume**) to the speaker with a PWM frequency of 62.5KHz, for half the duration of a single period of the **frequency** before pulling it `LOW`. (Using Timer1 compare-match interrupts to maintain the input frequency) This high-speed PWM is beyond your range of hearing, (and probably the functioning range of your speaker) so it will just sound like a quieter or louder version of the input frequency!

**vol.fadeOut**(unsigned int **duration**);

This will cause the currently playing tone to fade out over the **duration** specified in milliseconds.

**vol.noTone**();

This is identical in function to the standard `noTone()` function, this stops any currently playing tones.

**vol.delay**();   **vol.delayMicroseconds**();
**vol.millis**();   **vol.micros**();

These are replacements to the standard time-keeping Arduino functions designed to compensate for the changes in the Timer0 clock divisor. See [Limitations](#limitations).

**vol.end**();

This stops any currently playing tones, and resets Timer0 to it's default functionality. Creative use of `vol.begin()` and `vol.end()` can usually resolve conflicts with other libraries or functions that might need Timer0 (volume) or Timer1 (frequency) to be in their usual settings.

**vol.alternatePin**(bool **enabled**);

This causes the AVR to use the ALTERNATE_PIN defined in the [Supported Pins](#supported-pins) section for sound production.

----------
# Supported Pins

By default, the library uses DEFAULT_PIN for the speaker, *(changes from board to board due to Timer0 channels)* but if you need this pin for digitalWrite's, you can call *vol.alternatePin(**true**)* to use ALTERNATE_PIN instead.

| Board                           | DEFAULT_PIN | ALTERNATE_PIN | Tested |
|---------------------------------|-------------|---------------|--------|
| (**Uno**) ATmega168/328(pb)     | 5           | 6             | YES    |
| (**Mega**) ATmega1280/2560      | 4           | 13            | YES    |
| (**Leo/Micro**) ATmega16u2/32u4 | 9           | 10            | YES*   |

*I recently killed my only ATmega32u4 board while stripping it for low-power usage and don't have one to test current releases of the library. If anyone who has a working one wants to report compatibility back to me, please do so as I've only tested the initial release!

----------
# Limitations
Unfortunately, cheating the Arduino's normal functions in this way means we'll lose some of them. This is also still a proof-of-concept library at this point, so it may break more functionality than I'm aware of. Sorry!

~~**16MHz Only:**~~

~~I haven't programmed in options for 8MHz boards yet, though if you want to use one, just replace all occurrences of "16000000" in the library files with "8000000" and it may work.~~

Automatic detection of CPU speed was added in version 1.0.2!

**ATmega* Only:**

I don't know if I'll have this working on ATTiny*5 boards any time soon, though it's theoretically possible on any AVR with >= 2 timers. For now, it's only confirmed working on Arduino Uno (ATMega168/328) and Mega. (ATMega1280/2560)

**Volume is limited to pins ~~5 & 6:~~**

This is because only pins ~~5 & 6~~ are driven by Timer0, *which can do PWM at a frequency higher than your hearing range!* This is the main trick behind the volume function. It also means that while you're using Volume, normal `analogWrite()` use probably won't work on these two pins.

*Now that the Mega168, 328, 1280, 2560, 16u2/32u4 and more are now supported, the supported pins differs from board to board. See [Supported Pins](#supported-pins) section.*

**Volume alters Timer0 for 62.5KHz PWM:**

Speaking of Timer0 - it's normally used for the `delay()`, `delayMicroseconds()`, `millis()` and `micros()` functions. Normally with Timer0 set with a divisor of 64, `delay(1000)` would wait for 1 second - but because Volume sets Timer0 with a divisor of 1, `delay(1000)` will now only wait for 15.625ms! But don't worry. Volume provides alternative `vol.delay(time)` and `vol.delayMicroseconds(time)` functions with the math fixed for you. This new divisor is necessary to drive PWM at 62.5KHz, faster than you can hear.

~~**Volume does not yet offer fixed millis() or micros() functions:**~~

~~I haven't gotten around to toying with these yet. If you need to use `millis()` or `micros()` BETWEEN playing sounds, just use a `vol.end()` to reset Timer0 to it's default function, and `vol.begin()` to use it for Volume again after you're done.~~

Version 1.1.1 added proper millis() and micros() support! See [Functions](#functions).

----------
# Contributing
As I've only written one library before for my own use, I'm still new to this! Any advice or pull requests are welcome. :)

----------
# License and Credits
**Developed by Connor Nishijima (2016)**

**Special Thanks to Andrew Neal** (For putting up with the incessant and inconsistent artificial "cricket-duino" hidden in his vent that I developed this library for.)

**Released under the [GPLv3 license](http://www.gnu.org/licenses/gpl-3.0.en.html).**
