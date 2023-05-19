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

Option 1 is the route we are going to go as we are interested in the bare minimum. Option 2 would require multiple memory chips on top of an additional `controller` chip to switch between the different memory chips. An example of a game that is 32 kilobytes is the original Tetris game from 1989, while some of the more complex games such as the pokemon games utilize multiple memory chips to account for much more storage. More information on Memory Banking and the different types of cartridges as well as which games required what mappings can be found [here](https://gbdev.gg8.se/wiki/articles/Memory_Bank_Controllers)