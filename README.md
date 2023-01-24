# ATTINY402_bringup
notes on hardware setup and programming an attiny402 using arduino IDE


## Intro

I recently needed a oneshot timer with an open-collector / active-low output.  
I could certainly do this with a 555 and an inverted, but I'm badging this into an existing product and wanted to keep part count / BOM cost to a minimum.

Maybe there's another more elegant way to get this done w/ minimal parts, but in frustration I decided to just use a controller.

I needed a small one, that was in-stock (parts shortage is still a thing), and I'm cheap.

I saw the ATTiny85 was quite popular w/ much prior art in the arduino space, but they're all >$1.5 each in small quantities.

I noticed there were attiny variants in the <$0.50 realm, that seemed reasonable to me, especially if it could be my sole component in the BOM.  

Just need to make sure I can program it w/o any special tools, and software support is easy. Arduino swim lane seems the path of least resistance here.    
After some research online I came across the [megaTinyCore](https://github.com/SpenceKonde/megaTinyCore) project, which seemed to have support for the [Attiny402](https://github.com/SpenceKonde/megaTinyCore/blob/master/megaavr/extras/ATtiny_x02.md). I hastily pulled the digikey trigger ahead of a work trip.

On my return parts were in hand, now to actually get things working...  
The documentation in the megaTinyCore project is... extensive. There's a lot of infomration, and really all I cared about was the following. So hopefully these cliff notes are useful to some future traveler:


### Install tools:

There's a bunch of info here on why all the different Arduino IDEs are bad mkay.   
For reasons unknown I was getting certificate errors when trying to install in a number of 1.8.x IDEs (probably due to local machine settings).   

* I eventiually just gave up, installed Arduino 2.0.3
* Added `http://drazzy.com/package_drazzy.com_index.json` in File > Preferences 
* And Installed 'megaTinyCore' through the board manager tab. 

This seems to work fine for me in practice...

### Create UPDI Programmer

TODO: document wiring

https://github.com/SpenceKonde/AVR-Guidance/blob/master/UPDI/jtag2updi.md

### Set programmer/board settings / bootloader flash

* Tools > select COM4 or whatever COM port the flasher enumerates as
* Tools > Board > megaTinyCore > `ATtiny412/402/212/202`
* Tools > Chip > `402`
* Tool > Programmer > `Serial UPDI - SLOW: 56700 baud`

Plug the 402 into GND, 5V, and wire the UPDI pin (6) to the RX pin on the programmer.

* Tools > Burn Bootloader

### Flash simple sketch:

```c
#define OUTPUT_PIN 2

void setup() {
  pinMode(OUTPUT_PIN, OUTPUT);
}

void loop() {
  //blink test code
  digitalWrite(OUTPUT_PIN, LOW);
  delay(1000);
  digitalWrite(OUTPUT_PIN, HIGH);
  delay(1000);
}
```

* Pressing the program button doesn't work! you'll get an error: `A programmer is required to upload`
* To program: `Sketch` > `Upload using programmer`

  Surprisingly this seems to work without needing to put the device through a reset state, which is actually very much appreciated for development!

```
Sketch uses 320 bytes (7%) of program storage space. Maximum is 4096 bytes.
Global variables use 10 bytes (3%) of dynamic memory, leaving 246 bytes for local variables. Maximum is 256 bytes.
SerialUPDI
UPDI programming for Arduino using a serial adapter
Based on pymcuprog, with significant modifications
By Quentin Bolsee and Spence Konde
Version 1.2.3 - Jan 2022
Using serial port COM4 at 57600 baud.
Target: attiny402
Set fuses: ['0:0b00000000', '2:0x02', '6:0x04', '7:0x00', '8:0x00']
Action: write
File: C:\Users\jcorcoran\AppData\Local\Temp\arduino-sketch-3E81EEF084FE66A73AB488FD67188425/tiny402_oneshot.ino.hex
Pinging device...
Ping response: 1E9227
Setting fuse 0x0=0x0
Writing literal values...
Verifying literal values...
Action took 0.04s
Setting fuse 0x2=0x2
Writing literal values...
Verifying literal values...
Action took 0.03s
Setting fuse 0x6=0x4
Writing literal values...
Verifying literal values...
Action took 0.03s
Setting fuse 0x7=0x0
Writing literal values...
Verifying literal values...
Action took 0.03s
Setting fuse 0x8=0x0
Writing literal values...
Verifying literal values...
Action took 0.03s
Finished writing fuses.
Chip/Bulk erase,
Memory type eeprom is conditionally erased (depending upon EESAVE fuse setting)
Memory type flash is always erased
Memory type lockbits is always erased
...
Erased.
Action took 0.01s
Writing from hex file...
Writing flash...
[                                                  ]
[==========                                        ] 1/5
[====================                              ] 2/5
[==============================                    ] 3/5
[========================================          ] 4/5
[==================================================] 5/5
Action took 0.14s
Verifying...
Verify successful. Data in flash matches data in specified hex-file
Action took 0.07s
```
