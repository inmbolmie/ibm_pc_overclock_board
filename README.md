# IBM 5150/5160 Overclock Board

Schematics for an IBM 5150/5160 overclock board to get CPU speed up to 8.33Mhz

![socket](/images/img8.jpg)

Originally shared here: https://www.vcfed.org/forum/forum/genres/pcs-and-clones/72067-ibm-5160-overclock-sergey-s-way

In that thread Sergey Malinov posted a proposal of overclocking circuit for an IBM 5160 based on his design for Sergey's XT system. I was intrigued as it seemed a very fun project to try to implement and test, but I haven't found any evidence of anyone implementing this particular circuit in practice (I know there are some successes with similar devices like PC-Sprint here http://www.vcfed.org/forum/showthread.php?60803-I-overclocked-my-IBM-5150). It's not even published in Sergey's website.

So I did it, and with some minor modifications over Sergey's proposal I have been able to get my 5160 to 8.3Mhz for a 93% performance improvement. So it works, and very well indeed.

his is the original schematic:

http://www.vcfed.org/forum/attachment.php?attachmentid=17804&d=1395123082

And this is my modified schematic.

[schematics](turbo8088%20v2.pdf)

Things that I’ve changed from Sergey’s schematic:

* As Sergey says in the original thread, the wiring in the 74LS92 counter used to generate PCLK is incorrect. I’ve changed it to generate a clock of 1/3 OSC frequency. Also I see no reason to wire this counter to the reset circuit so I removed that connections.
* There is no separate reset circuit, everything is connected to the system RESET lines. The reason is that I found that keyboard initialization is picky about reset timing, and using the separate circuit often ended in 301 POST errors and a non-functional keyboard. The original reset signal is related to a signal that comes from the power supply that tells the system that its power output is stable enough to run (PWR_GOOD).
* I’ve added a debouncing circuit to the TURBO input signal. This is based on a 74LS14 (Schmitt trigger inverter). The signal comes from a mechanical switch, so it is much cleaner to pass it through the debouncer.

Everything else is as specified in Sergey’s design.

For the installation the difficult part is interfacing this circuit with the 5160 mainboard. The circuit replaces the 8284 clock generator that is just next to the main 8088 CPU. You can be lucky and get a mainboard that has its 8284 socketed, or you can be unlucky (like me) and have the IC directly soldered to the board. If socketed, you simply unplug the 8284 and replace it with a custom built cable that goes pin-to-pin to the overclocking circuit. If soldered, you have to disassemble the entire system to remove the mainboard, unsolder the 8284 and solder a proper DIP18 socket in its place. This can be a real pain if you don’t have the proper equipment.

![socket](/images/img1.jpg)

Regardless of needing or not the 8284 desoldering operation, if you want to go high in overclocking you will need at least to remove the floppy drive and hard disk to get access to the needed DMA signals (HRQDMA and DMAWAIT) that will disable TURBO operation when DMA activity is detected. If you don’t, you will first have unreliable DMA operation (hard disk errors) and if you go high enough the system will not even get to POST. As an example in my case hard disk operation becomes unreliable with an oscillator over 20MHZ and won’t post over 22MHz. Up to 20Mhz I had no DMA issues even without the signals connected, but other systems could be less permissive.

This is the location of those signals in my mainboard. I got the locations from the IBM 5160 technical reference guide:

* HRQDMA from pin 10 of the 8237 DMA controller (U28 )
* DMAWAIT from pin 7 of a 74LS175 flipflop (U88 ). It seems that can also be obtained inverting pin 4 from the 8284 as it should be connected to pin 6 of U88, but pin 7 gives it not inverted.

![mainboard_signals](/images/img2.jpg)

I built a custom pint-to-pin DIP18 to DIP18 cable using an old IDE ribbon cable, two DIP18 ribbon connectors and two DIP18 turned sockets. It takes 5 minutes to make it. The cable length is approx. 20 cm.

![cable](/images/img3.jpg)

My ribbon connectors are DIP20 but only because that was what I had readily available, they work well with a DIP18 turned socket.

The first prototype I made was using a breadboard. This version has a 20Mhz clock generator and lacks the antibouncing circuit

![breadboard](/images/img4.jpg)

Then for the final circuit I used a soldered prototype board. Good enough for the moment, cheap and can be made very quickly.

![prototype](/images/img9.jpg)

![prototype_back](/images/img5.jpg)

![general](/images/img6.jpg)

Finally a toggle switch for better retro looking to enable and disable turbo mode, installed drilling an empty ISA bracket

![switch](/images/img7.jpg)

Some details and “lessons learnt” through the process:

* Used an 8Mhz rated V20 processor in place of the original 8088. V20 is not required, but gives some extra performance and enables use of applications that require the 80186 instruction set.
* Removed entirely my 8087 coprocessor as it gave me many stability issues even at stock speed. Didn’t even bother to get a higher rated part, as I have no particular interest in running Lotus 1-2-3 overclocked or something.
* With the original 8284 clock generator I couldn’t get over 20Mhz. I replaced it for an AMD part I got from Ebay. With this I can go up to 25Mhz oscillator speed. My local electronics shop sold me as a replacement a NEC 71084 (supposedly pin compatible with 8284), but it doesn’t work with this design because it doesn’t drive the OSC output when F/C signal is high. The real 8284 drives OSC always from the crystal input frequency regardless of the F/C signal state.
* I have a late revision 64-256Kb board and a 384KB ISA expansion card. Mostly 150ns 4164 ram modules. No particular issues with onboard RAM to get to 25Mhz. The expansion card is a different history, as for going over 24Mhz I had to replace some 150ns modules with 120 ns modules. The replaced 150ns modules just work fine if I place them in the mainboard banks, but fail miserably if placed in the expansion card. I imagine that expansion card memory has some more strict timing issues that mainboard memory.
* Over 24Mhz I have some memory corruption and graphics glitches in my EGA card (paradise PEGA2A) unless I place a cooling fan in front of it. It’s not strictly an overheating problem because if I switch off overclocking it instantly works fine. No problem at 24Mhz though. I haven’t tried using a CGA card (can get a look if someone is interested).


As said, the faster I can get is 25Mhz OSC, equivalent to 8.33Mhz CPU clock speed. Around that speed everything seems to begin falling apart. Checkit reports CPU speed as 7.91Mhz because I imagine there is some background DMA activity that reduces slightly the effective clock speed. No problem at that speed to pass any memory or integrity test, although the graphics card needs some extra cooling for glitchless operation.
