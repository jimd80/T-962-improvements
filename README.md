T-962 reflow oven improvements
==============================
Custom firmware for the cheap T-962 reflow oven utilizing the _existing_ controller hardware.

 - [Wiki] for more info
 - [Hackaday post]
 - We have [Travis-CI] in place to build pull requests

### Introduction

As we had use for a small reflow oven for a small prototype run we settled for the `T-962` even after having seen the negative reviews of it as there were plenty of suggestions all across the Internet on how it could be improved including replacing the existing controller and display(!). After having had a closer look at the hardware (replacing the masking tape inside with Kapton tape first) it was obvious that there was a simple way to improve the software disaster that is the T-962.

### Extra software improvements

All credits go to the amazing work of UnifiedEngineering. This fork adds some extra's. A letter suffic is added to the upstream version to indicate the customisations.

Changed in v0.5.2d:
 
 - Added vscode files and flash script
 - Bugfix: tc offset and gain only effective after start reflow / reset
 - Bugfix: manual mode not reset after re-entry (pressed stop)
 - Bugfix: setting TC gain <= -10 causes crash (reboot) due to format (use 00.0 instead of 0.00)
 - Improvement: Slower 230v pwm (fan+heat) 2 Hz instead of 5 Hz for less net noise
 - Improvement: show TC temp during setup for easier tuning of tc gain and offset
 - Improvement: TC gain extended to -50..+50 instead of -25..+25
 - Improvement: Bake mode goes back to preheat if regulation is 1 degree off. Changes to 5 degrees to allow some regulation swing
 - Visual: Use same font in bitmaps (especially smaller S-button)
 - Visual: Removed dots in graph and added "degrees c" and "s" axis-labels (looks more clear, personal preference)
 - Visual: Main: Renamed manual/bake mode to manual
 - Visual: Setup: Replaced < > to up and down arrow
 - Visual: Manual: Renamed thermocouple R to FNT (front) and L to BCK (back) and cold juction to CPU
 - Visual: Manual: Added : between temperature label/value
 - Visual: Reflow: Renamed RUN to TIME in reflow screen and put on top
 - Visual: Reflow: added degrees symbol in act and set values,
 - Visual: Reflow: Added progress bar,
 - Added popular Chipquick profiles (TS series, without need for refridgeration)
 - keep reflow screen open when done to check chart
 - Visual: Removed [S] from about screen
 - Serial interface 57600 baud instead of 115200 to allow one tool for flashing and terminal using same baud rate
 - Visual: custom logo
 - changed standby temp to 40 instead of 50 degrees
 - show CPU temp on main screen
 - added alarm when CPU is overheating
 - added debug function when pressing F3 on about screen
 - disabled system fan PWM, too much jitter. toggle between full on and off depending on cpu and oven temperature

### TODO list
 
 - todo: rename act to oven when done
 - todo: hysteresys system fan
 - todo: reflow done beep when <40 degrees

### Hardware improvements

Here are a few improvements made to the cheap T-962 reflow oven utilizing the _existing_ controller hardware with only a small, cheap, but very necessary modification. As you have to open the top part of the oven anyway to reflash the software this is a no-brainer fix:

#### Replace stinky masking tape

Instructable suggesting [replacing masking tape with kapton tape](http://www.instructables.com/id/T962A-SMD-Reflow-Oven-FixHack/?ALLSTEPS).

#### Cold junction compensation

The existing controller makes the assumption that the cold-junction is at 20 degrees Celsius at all times which made keeping a constant temperature "a bit" challenging as the terminal block sits _on_top_of_an_oven_ with two TRIACs nearby.
We can fix this by adding a temperature sensor to the connector block where the thermocouples are connected to the controller board.
It turns out that both an analog input and at least one generic GPIO pin is available on unpopulated pads on the board. GPIO0.7 in particular was very convenient for 1-wire operation as there was an adjacent pad with 3.3V so a 4k7 pull-up resistor could be placed there, then a jumper wire is run from GPIO0.7 pad to the `Dq` pin of a cheap [DS18B20] 1-wire temperature sensor that gets epoxied to the terminal block, soldering both `Vcc` and ground pins to the ground plane conveniently located right next to it. Some hot-glue may have to be removed to actually get to the side of the connector and the ground plane, someone seems to have been really trigger-happy with the glue gun!

[Wiki: cold junction compensation mod](https://github.com/UnifiedEngineering/T-962-improvements/wiki)


#### Check mains earth connection

As mentioned elsewhere, make sure the protective earth/ground wire from the main input actually makes contact with the back panel of the chassis and also that the back panel makes contact both with the top and bottom halves of the oven!

#### System fan PWM control

The system fan is very noisy an can be turned of most of the time. The custom firmware uses spare `ADO` test point to control it.

[Wiki: system fan PWM mod](https://github.com/UnifiedEngineering/T-962-improvements/wiki/System-fan-control)

### New firmware

The firmware was originally built with LPCXpresso 7.5.0 as I've never dealt with the LPC2000-series NXP microcontrollers before so I just wanted something that wouldn't require TOO much of work to actually produce a flashable image. Philips LPC2000 Flash Utility v2.2.3 was used to flash the controller through the ISP header present on the board.

LPCXpresso requires activation but is free for everything but large code sizes (the limit is larger than the 128kB flash size on this controller anyway so it's not really an issue). The flash utility unfortunately only runs on Windows but Flash Magic is an alternative (see Wiki for more flashing instructions).

With help from the community the project now also builds standalone using the standard `gcc-arm-none-eabi` toolchain, see `COMPILING.md` for more information.

The MCU in this particular oven is an LPC2134/01 with 128kB flash/16kB RAM, stated to be capable of running at up to 60MHz. Unfortunately the PLL in this chip is not that clever so with the supplied XTAL at 11.0592MHz we can only reach 55.296MHz (5x multiplier). Other variants exist, the [Wiki] has more information about this.

wiki: [Flashing firmware]

### Contributing
This is mainly tested on a fairly recent build of the T-962 (smallest version), build time on the back panel states 14.07 which I assume means 2014 July (or less likely week 7 of 2014), success/failure reports from other users are welcome!

This is very much a quick hack to get only the basic functionality needed up and running. Everything in here is released under the GPLv3 license in hopes that it might be interesting for others to improve on this. Feedback is welcome!

Happy hacking!

# Acknowledgements
This project is using the [C PID Library - Version 1.0.1, GPLv3]

[wiki]: https://github.com/UnifiedEngineering/T-962-improvements/wiki
[Travis-CI]: https://travis-ci.org/UnifiedEngineering/T-962-improvements
[Flashing firmware]: https://github.com/UnifiedEngineering/T-962-improvements/wiki/Flashing-the-LPC21xx-controller
[DS18B20]: http://datasheets.maximintegrated.com/en/ds/DS18B20.pdf
[hackaday post]: http://hackaday.com/2014/11/27/improving-the-t-962-reflow-oven/
[C PID Library - Version 1.0.1, GPLv3]:https://github.com/mblythe86/C-PID-Library
