---
layout: posts
title:  "SDR Project Phase 1: Oscillator"
date:   2024-02-12
categories: sdr_project_log
author_profile: true
---

## An aside
One thing a radio needs is a name.  Names can be cool, like the HackRF, or uninspired, like the Icom IC-730.  I would like a cool name for my radio, but one that doesn't oversell its capabilities.  The name can't also be taken, so PicoSDR is out.  I'm thinking about calling my radio design the OscarSDR.  There is a double meaning to this- OSCAR is an acronym in the amateur radio community that means Orbiting Satellite Carrying Amateur Radio, and perhaps OscarSDR will be capable of receiving them (and transmitting to them with the help of a power amplifier).  But also, Oscar the Grouch is a character on the TV show Sesame Street who lives in a trash can, and OscarSDR is made out of the cheapest parts that will get the job done.  So from now on, this radio will be called OscarSDR.

## The heart of a radio
In nearly all radios, there is a variable frequency oscillator (VFO) that controls downconversion from the radio frequency (RF) to an intermediate frequency (IF) or directly to baseband (the frequency of the signal of interest).  Sometimes the VFO is also called the local oscillator (LO), which is a term used for the frequency source inside a radio that is mixed with the RF for downconversion.  The term LO does not require that the clock source be able to generate a range of frequencies, but the term VFO implies that the frequency can be controlled.

OscarSDR needs a VFO that is capable of frequencies up to about 150 MHz, and because it does quadrature down conversion, it needs to be able to produce two frequencies that are 90 degrees apart in phase.  Additionally, OscarSDR needs a clock source capable of driving the Tayloe detector at the intermediate frequency of about 1 MHz.  It turns out that there is a convenient part called the SI5351 which produces three clock outputs and can control the phase offset between clocks with identical frequency.  A single SI5351 is enough to be a clock source for the entire receiver chain, and with a few switches, it can also be the source for the transmitter chain.  On Amazon, SI5351 boards are under $10 and offered by multiple sellers, so this is a perfect candidate for OscarSDR.

## Not so fast
Like many things in life, part 1 of this SDR series was not without struggles.  My first order was for an SI5351 board that was made in China with a purple PCB with the "brand" CJMCU on the back of it.  Many different sellers sell this board for around $8.  However, I could not get the first board to work- it put out some kind of signal, but it was very jittery.  So I ordered a second board, and it was jittery too (so bad I can't show it in a picture).  At this point, I wondered if I was doing something wrong, because what is the likelihood that I would get two parts bad in a row?  I spent hours trying multiple libraries and even started trying different combinations of 5V and 3.3V power supplies and logic levels.  Nothing worked.  Here's a picture of the bad board, so you know what to avoid.

![The top of the bad board](/assets/images/phase1/BadBoard1.png)

![The back of the bad board with the CJMCU branding](/assets/images/phase1/BadBoard2.png)


Since I needed to get the clock source working to build the rest of the radio around, I ordered a three-pack of SI5351 boards from another seller, and only one of them worked right.  But that confirmed to me that it wasn't my mistake that the first two didn't work, and I had wasted several hours troubleshooting broken parts.  If you're keeping track, I am up to five SI5351 boards and only one worked right.  After gaining confidence that I was giving them the proper power supply and right control signals (we'll talk about I2C in a part about the software or the Pi Pico I suppose), I went back and tried to diagnose the problem with other boards.  The first two of the boards had a bad crystal, so the SI5351 was making nonsense (the jittery signal mentioned earlier).  One of the boards in the three-pack had one bad clock output, but two of them seemed fine- but this is one click too few for OscarSDR.  One of the boards in the three-pack did not have a functioning I2C bus.  For the reader to keep track, the purple SI5351 boards have a 20% success rate by my estimate.  The picture below is what a dead crystal looks like on an oscilloscope.

![No crystal output](/assets/images/phase1/NoClock.jpg)

Just for fun, and since it had been such a long time, I ordered some 25 MHz crystals from Amazon and replaced one of the bad crystals.  Just like that, one of the bad SI5351 boards worked just fine.  So these boards can sometimes be repaired, but I don't think that it's worth the effort.  Below is what a good crystal output looks like with all the distortion from being present in a circuit and being probed by the oscilloscope.

![Working crystal output](/assets/images/phase1/GoodClock.jpg)

## A light in the darkness
The SI5351 boards with a purple PCB and the CJMCU marking on the back are not the only supply available.  The purple PCB is a clone of a design created by Adafruit.  The Adafruit version is available through their website without free shipping or a few dollars more on Amazon.  At first, I did not see the need to spend the extra money, but for just a couple of dollars more you can have a board that uses good quality parts and not parts that failed their quality inspection.  I ordered two of the blue Adafruit branded SI5351 boards and they both work perfectly.  I have settled on using the Etherkit SI5351 library in Arduino to get the Pi Pico to control the SI5351, as it has all the detailed functions I need to command the SI5351 properly.  Below is what I was aiming for- two clock outputs at the same frequency but 90 degrees apart in phase.  This can drive quadrature down conversion!

![Success](/assets/images/phase1/success.jpg)

![This one is worth the money](/assets/images/phase1/GoodBoard.png)

## The first step of many
At this point, I don't have much to show.  Being the core of the radio, there isn't any testing to do other than verify that the correct frequencies are being created at the right phase.  In future parts, I will experiment with different configurations and tweak resistor and capacitor values to get everything working correctly.

## Extra steps for higher performance
Right now, the SI5351 in OscarSDR is not calibrated. I am relying on the accuracy of the 25 MHz crystal to generate accurate frequencies.  If I can get my hands on a good frequency reference, I will be able to calibrate the frequency reference and get a more accurate radio.  The SI5351 also generates a clock signal which is a square wave, not a sine wave.  A quick and dirty radio like OscarSDR can use the clock source as is, but a better radio would put a low pass filter on the VFO to make sure it is mixing a sine wave and not generating a bunch of spurious emissions.
