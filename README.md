# ttyMIDI

ttyMIDI is a GPL-licensed program that allows external serial devices to
interface with the ALSA sequencer.

The original code comes from here: http://www.varal.org/ttymidi/

COMPILATION

The ttyMIDI source code is comprised of a single C file.  To compile it, just
run the following command:

	make

This program depends on libasound2, so you should have the development headers
for that installed. In Debian or Ubuntu, you can install it by running:

	apt-get install libasound2-dev

After you compile ttyMIDI, you may wish to copy it to /usr/bin for easy
access. This can be done simply by following command:

	sudo make install

USAGE

First, you need an external device that can send MIDI commands through the
serial port.  To find out more about programming an external device, read the
TTYMIDI SPECIFICATION section.  If you are using an Arduino board
(http://www.arduino.cc), read the instructions under the arduino folder.  Once
your device is programmed and connected to your PC's serial port, follow the
instructions below.

To connect to ttyS0 at 2400bps:

	ttymidi -s /dev/ttyS0 -b 2400

To connect to ttyUSB port at default speed (115200bps) and display information
about incoming MIDI events:

	ttymidi -s /dev/ttyUSB0 -v

ttyMIDI creates an ALSA MIDI output port that can be interfaced to any
compatible program.  This is done in the following manner:

	ttymidi -s /dev/ttyUSB0 &         # start ttyMIDI
	timidity -iA &                    # start some ALSA compatible MIDI program
	aconnect -i                       # list available MIDI input clients
	aconnect -o                       # list available MIDI output clients
	aconnect 128:0 129:0              # where 128 and 129 are the client numbers for
                                      # ttyMIDI and timidity

Further, ttyMIDI creates an ALSA MIDI input port that feeds incoming MIDI events
back to the serial port. Before better documentation exists, check the header file of
the ardumidi library to figure out how to read this data at the Arduino end.

If you would like to use a GUI to connect your MIDI clients, there are many
available.  One of my favorites is qjackctl.


TTYMIDI SPECIFICATION

The message format expected by ttyMIDI is based on what I could gather about the
MIDI specification online.  I tried to make it as similar as possible to the
specification, but some differences exist.  The good news is that as long as you
follow the specification described below, everything should work.

Every MIDI command is sent through the serial port as 3 bytes.  The first byte
contains the command type and channel.  After that, 2 parameter bytes are
transmitted.  To simplify the decoding process, ttyMIDI does not support
"running status", and it also forces every command into 3 bytes.  So even
commands which only have 1 parameter must transmit byte #3 (transmitting a 0 in
this case).  This is described in more details in the table below:

byte1       byte2                     byte3                     Command name

0x80-0x8F   Key # (0-127)             Off Velocity (0-127)      Note OFF
0x90-0x90   Key # (0-127)             On Velocity (0-127)       Note ON
0xA0-0xA0   Key # (0-127)             Pressure (0-127)          Poly Key Pressure
0xB0-0xB0   Control # (0-127)         Control Value (0-127)     Control Change
0xC0-0xC0   Program # (0-127)         Not Used (send 0)         Program Change
0xD0-0xD0   Pressure Value (0-127)    Not Used (send 0)         Mono Key Pressure (Channel Pressure)
0xE0-0xE0   Range LSB (0-127)         Range MSB (0-127)         Pitch Bend

Not implemented:
0xF0-0xF0   Manufacturer's ID         Model ID                  System

Byte #1 is given as COMMAND + CHANNEL.  So, for example, 0xE3 is the Pitch Bend
command (0xE0) for channel 4 (0x03).

