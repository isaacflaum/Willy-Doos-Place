---
layout: post
title:  "Simple Gameboy Cartridge"
date:   2023-05-15 15:23:09 -0400
categories: update
---
After seeing people create their own custom gameboy cartridges - such as [Sebastian's Wifi Gameboy Cartridge](https://there.oughta.be/a/wifi-game-boy-cartridge) or [HDR's Custom Gameboy Camera Cartridge](https://github.com/HDR/Gameboy-Camera-Flashcart) I wanted to dip my feet into the pool. Unlike all these complicated boards with tons of resistors, capacitors, and microcontrollers - I want to figure out the bare minimum needed to run a gameboy game. And it turns out it's not much.

A gameboy cartridge has 32 pins exposed to the gameboy in the form of an edge connector that is plugged into the Gameboy's cartridge slot. 24 of those 32 pins are for accessing memory with 16 of those 24 pins demarking the address to read or write while the last 8 of those 24 pins contain the byte of data being read or written. The rest of the pins are a clock cycle, read and write enable and an audio pin. Of course there are also 2 pins for +5V power and ground.

Because there are 16 pins for the address location - this gives us 2^16 or 65,536 different one-byte cells of data to store information. Actually it's a bit worse than that - since the system reserves one of the 16 address pins, we really only have 32,768 different one-byte cells or 32 kilobytes of data. This means any game has to either:
    * be 32 kilobytes or smaller
    * have 2 or more memory chips (called memory banks) on the gameboy cartridge.

Option 1 is the route we are taking as we are interested in the `bare minimum`. Option 2 would require multiple memory chips on top of an additional `controller` chip to switch between the different memory chips - yikes! An example of a game that is 32 kilobytes is the original Tetris game from 1989, while some of the more complex games such as the pokemon games utilize multiple memory chips to account for much more storage. More information on Memory Banking and the different types of cartridges as well as which games required what mappings can be found [here](https://gbdev.gg8.se/wiki/articles/Memory_Bank_Controllers)

So you've gotten this far and might be asking but wait - isn't a gameboy game just data stored on a memory chip - don't we only need one component, a memory chip, to have a fully functioning gameboy cartridge? And the answer is not really, as memory chips all have different pin layouts, different read and write access times, and even different voltages and currents that must be steadily maintained or the chip can have non-deterministic behavior - this is bad! This means we will probably also need current limiting resistors and current steadying capacitors in our circuit design. Boo!

Before I ramble on to death, it turns out there are many memory chips in theory that have fast enough read/write access times and don't care too much about steady voltage and current - great! ....Buuuuuut due to the chip shortage the actual list is much smaller. For this project I decided to go with the `SST39` series of flash chips which are relatively cheap (under 5$ a pop as opposed to the other EEPROM chips that fit the bill but are $10+ a pop) and available (mouser lists the SST39 chips as `regularly stocked` on their website - excellent). The `SST39` series of chips also reads and writes 8 bits at a time just as our gameboy does so we can map the data pins on the cartridge directly to the data pins on the `SST39`! Same with the address pins, we can use the first 16 address pins on any of the SST39SF0`x`0 chips and map them directly to the 16 address pins on the gameboy cartridge. From there we have a few more pins to supply power to and then a couple of ground pins and we are good to go. As for only using one chip for our gameboy cartridge this is where we have to cave. One pin of the `SST39` requires a 12Kohm resistor. I also added a parallel 10uF capacitor to the power supply loop to stop a bit of humming from electrical feedback. It turns out you can have a fully functioning gameboy cartridge with only 3 components - a resistor, capacitor and memory chip! 2 if you don't mind humming but I thought is is better design to not have humming!

Once the prototype was working, I designed a circuit board, had it fabricated by JLCPCB and PCBWay and began soldering. The capacitor and resistor are easy enough to solder on but the memory chip has 32 small pins. It is much easier to solder on an adapter and then plug the chip right into the adapter socket. The socket will also let us reprogram the chip if we wish to put a different gameboy game on the cartridge.

You can check out the project [on github here](https://github.com/isaacflaum/flashable-dmg-cartridge). It turns out you can have a fully-functioning gameboy cartridge with only 3 components on the board - pretty cool!

And here is what everything ended up looking like:

### Socket for the memory chip
![Here's the board](/images/simple-cartridge/icsocket.webp)

### Resistor and Capacitor
![Here's the board](/images/simple-cartridge/passives.webp)

### The Whole Cartridge
![Here's the board](/images/simple-cartridge/cartridge.webp)

### The Final Result
A custom `game` I created and uploaded to the memory chip to show it works. I used the TL866II programming adapter along with a PLCC-32 to DIP socket adapter to write the gameboy game to the memory chip. It's not actually a game but just a logo I quickly whipped up in GIMP as a proof-of-concept for the awesome Factory 3 maker-space in Portland.
![Here's the game](/images/simple-cartridge/finalresult.webp)