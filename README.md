# QuickJoy-M-5-SV-201
## Introduction
The QuickJoy-M-5-SV-201 joystick is a peculiar PC "analog" joystick, connected via so called Gameport (nowadays obsolete). In fact, it is digital - the stick movements recognize only in one in 4 or 8 (diagonal) movements, but the interface to the PC is kind of analog - at least it works with standard 2 axis, 2 button drivers for analog joystick, but it uses quite a different principle of operation. This different principle has one benefit - calibration is done with one potentiometer.

## Gameport
Block diagram from pctim003.zip - "FAQ / Application notes: Timing on the PC family under DOS" by Kris Heidenstrom (kheidens@actrix.gen.nz) Version: 19951220, Release 3. Not sure on what license this document has been released, in 1995 people used to use the wording "public domain". I don't even think the mail adress is still valid. :
![Gameport Block Diagram](/gameport_block_diagram.png)

A standard PC joystick has 2 potentiometers, one for each axis. The PC doesn't read the value using a regular Analog to Digital Converter, instead it uses a 558 timer (Quadruple 555 timers) configured as monostable generators. On the schematic there is also a resistor (2K2) - for defining a minumim value (if the potentiometer would be set close to the VCC side, the capacitor would charge immediatelly and a capacitor (10n). This components vary in different implementations of the interface, so after connection of a new joystick a calibration is necessary (either by knob on the joystick or via software/drivers).
How the software interface works?
- the application/game writes to the gameport memory register a 0 to the bits 0 to 3 (or only on the selected bits/axis). This quickly shortens the related capacitors. The capacitors get charged with the current limited by the resistors (2K2) inside of the gameport and the joystick potentiometer axes. 
- the application/game cyclically reads out the register bits 0 to 3 to check if they changed to 1 - this would mean that the capacitors charged up. In the meantime the time is being measured (by the running software). The longer it takes to change the state from 0 to 1, the longer the capacitors take to charge, the more the joystick is from the "corner" position.
The gameport become obsolete among others because of the neccesity of the time measurement in the software. In times of MS Windows this has been handled by the driver, but in the DOS era this would have to be done by the game, which was time consuming (think of non-multitasking software at these times). At first simple CPU clock counting was used, but this become ineffectve when users used CPUs of different speed (think of moving from XT to 486 or even Pentium).

## Schematic
![Schematic](/QuickJoy_M-5_SV-201.svg)

The device is build up on 2 PCBs, one in the base, the other in the handle, connected together with several thin cables. For the user, apart from the regular fire buttons there are 2 potentiometer knobs and 3 switches. The potentiometers adjust the joystick calibration and the autofire frequency, the switches enable autofire for both fire buttons separatelly and exchange the button function (button1->button1 button2->button2 or button1->button2 button2->button1).

The joystick for work requires an interface, which generates proper signals on **3** axes. The X axis of the "joystick 2" is used to reset the timing circuitry. 

## Principle of operation of the M-5 Joystick
The U1 NE555 generator works in an astable configuration, the frequency can be adjusted with the PV1 potentiometer, which is essentially moving the centre point of the joystick (calibration). The generated pulses are being counted (or the frequency divided) by the 4024 binary counter. The counter gets reset, when the X axis of the "second" joystick (pin 11 of the gameport) is being pulled down to GND (awkward connection of T1, but this is how it is). When the Q5 output of the 4024 goes to high (2^5 pulses of the U1 generator) there is a pulse being generated on the base of T5/T6, which causes opening of the transistor and letting "some" current to the gameport. Then after the Q6 output switches to 1, the U1 generator stops, and the whole cycle can be only enabled by a new "0" edge on X axis of the "second" joystick. 
However, if the user moves the stick to a position of different than middle, the output pulses on T5/6 will come sooner or later. If the stick is is moved up/right, the output transistors are constantly open, so the gameport circuitry measures a very short time. If the stick is moved down/left, and Q5 high nothing happens (R5/6 is much higher than R7/8, so the 0 on Q6 will cause T3/4 not to open), but if the counter puts a 1 in Q6 the transistors T3/4 get open and cause a short pulse on T5/T6.

The U3 based generator is also astable and is used as autofire, with a frequency controlled by RV2. In the handle, the SW5 switch reverses the functions of the buttons, SW6 and SW7 enable/disable the autofire function on the front and back fire buttons separatelly. the D1 and D2 LEDs go on when the fire buttons are engaged.
