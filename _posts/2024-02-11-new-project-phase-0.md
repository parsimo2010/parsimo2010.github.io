---
layout: posts
title:  "New Project: An SDR- Phase 0"
date:   2024-02-11 19:00:00
categories: sdr_project_log
author_profile: true
---

Note: Sorry for some of the jargon in this post.  I will try to do better in future parts to define important terms and spell out acronyms the first time they are used.  If I get time I will try to go back in and be more descriptive in this post.

## A new radio project!
I'm starting a new project to design and build a radio, and I will log my progress here.  This is part zero, the description of the idea, the design constraints that I want to work within, and also my initial design concept.  Following this will be a series of parts where I describe the construction of each part in the order in which I accomplished them.  I'll probably have to change the design and add details from my initial draft shown here, so once everything is complete, I will try to publish a final design schematic with all the updates shown.

## The beginnings of a concept
I read on [RTL-SDR.com](https://www.rtl-sdr.com/an-hf-ham-radio-ssb-am-fm-cw-transmitter-made-from-a-raspberry-pi-pico-and-not-much-more/) an article about a guy who made a very simple software-defined radio (SDR) transmitter from a Raspberry Pi Pico and a few other components (but not much else).  The creator, Jon Dawson, has a website, [101things.com](https://101-things.readthedocs.io/en/latest/ham_transmitter.html), where he documents everything.  On his website, I found that he had also previously made a project of a simple SDR receiver with a Raspberry Pi Pico and similarly simple components.  These simple radios aren't nearly as capable as my BladeRF SDR which cost hundreds of dollars, but they are only about ten dollars worth of components.  I was amazed at the capability he squeezed out with that level of simplicity, and I wanted to do something similar.  

I didn't want to copy him, because where is the fun in that?  I have an amateur radio license, and have learned a lot about electronics, but have never built a radio from scratch, nor designed a radio.  I know from experience that I may think I understand something, but I never *really* understand it until I learn by doing.  So I resolved to design and build a radio of my own.  I also wanted it to be simple- maybe not as simple as Jon's design, but simplicity makes for easier understanding and probably a low cost.  One thing I noticed about Jon's design is that the Raspberry Pi Pico is an easily available part, but the analog switch he used is only available through electronics suppliers.  I think it would be really neat if my radio could be built entirely from "commodity" components that are easily available- think widely available parts on AliExpress or Amazon Prime.  One benefit of using parts from Amazon Prime is that you don't have to wait too long for shipping, so if you need to change your design you only need to wait two days.  Another thing I noticed about Jon's design is that it is restricted to being used at frequencies under 30 MHz- it is impressive that it works up to there, but I would prefer if I could work the popular 2-meter (144 MHz) amateur band with my radio.  So with that, here are my broad design constraints: 

- Receive and transmit up to 148 MHz to cover the 2-meter ham band
- Use only parts from Amazon Prime if possible
- Keep the cost under $200, and consider which features to sacrifice if the cost gets out of hand

## The draft design
I thought about this in my head for about two weeks and scoured Amazon to see if I could find good enough components, and found what I think are the key pieces, and I'm pretty sure the remaining pieces that aren't quite nailed down will be affordable and available, once I figure out exactly what I need.  The basic concept of this radio starts with the receiver chain.  There will be quadrature downconversion to a low IF and a Tayloe detector for a final downconversion step to baseband.  I came up with this concept in an attempt to avoid what I perceive as issues that some common designs present in designing a simple/cheap radio.  A superheterodyne radio without quadrature down conversion requires a lot of filtering for image rejection.  Designing all those filters would limit the flexibility of the radio, and it would take me longer to think through the filtering stages.  One way to fix the image problem is quadrature downconversion.  Images are still present, but they are 180 degrees out of phase, and having the information from the quadrature channel allows us to do image rejection with DSP.  Once you've decided to do quadrature downconversion, then almost immediately you think about a zero-IF design (at least I did).  A big problem with zero-IF designs is a DC offset, especially present when you use low-quality components, and I am worried about my components being low quality since I am sticking to a low budget.  So we can do quadrature downconversion to a low IF, then use a variant of the popular Tayloe detector to downconvert to baseband.  We cannot use the Tayloe detector directly because I don't think I can find any cheap multiplexers that switch as fast as 600 MHz (150 MHz x 4).  The key components that I picked are the AD831 RF mixer, of which we'll need two, an SI5351, which generates three clock signals for us and they can be phase controlled, and CD74HC4067 multiplexers because they are cheap.  I know I will need an op-amp for the Tayloe detector, but I have not picked exactly the one I want yet.  There will also need to be a collection of wires, cables, some prototyping board, resistors, capacitors, and a power supply.  Finally, a Raspberry Pi Pico will sample the baseband I and Q channels with its ADC.  

Then we need to consider the transmitter chain.  It will use PWM from the Pi Pico and a low pass filter to create I and Q channels, and a low pass filter to smooth it out to baseband.  Then that will be mixed with signals from the same SI5351 as the receiver chain into two more AD831 mixers,  The output will be combined into a single RF, which goes out to the antenna.  We can switch between receiving and transmitting to the antenna with another multiplexer.  There will probably need to be filtering, amplification, and bias voltages set around the design to make it functional.  We will add those things as needed in the individual development steps.  A schematic of my draft design is shown here
![draft SDR design schematic](/assets/images/Draft-Schematic.jpg)

## Cost estimates
These prices are taken all from the time of writing this, for components that are available with Prime.  This radio could be cheaper if you bought some things from AliExpress and waited a month for them to ship.  
- 4x AD831 boards @ $13 each: $62
- 1x SI5351 Clock Generator board (Adafruit): $10
 - Part 1 will discuss why you should pay the extra $2 and just get the Adafruit brand and not the Chinese ones with purple PCBs
- 5x CD74HC4067 16:1 multiplexers: $7 total
 - These were $6 for one and $7 for five, so I got that even though I don't need that many
- Raspberry Pi Pico Clone: $6
 - I got a version with 16 MB RAM and a USB-C connector, which is a little more expensive than the 4 MB RAM and Micro-USB that the original Pico had
- ElectroCookie 3x solderable breadboard: $12
- HUAREW IC kit with several different op amps: $20
- OSOYOO Electronic component kit with resistors, capacitors, and other stuff: $20
- SMA cables, male to male for connecting clock and mixers: $12
- Wires, soldering iron, solder: I already have it, but ballpark $20
- Multimeter, and an oscilloscope: $300
 - I am not counting this toward the price of the radio, but I don't think you'll get far if you don't have tools to troubleshoot
- Total price: $169

## Conclusion
So a total price of $169, plus I am expecting many hours of trial and error as I figure out how to get everything to work.  In the end, if I just wanted to play with radios, I would just buy one already built.  The amount of time I will spend doing this isn't worth the cost savings, and if I were to buy a radio I would get one professionally designed and have much better performance. I'm doing this because I want to be able to say that I made a radio of my own design.  Stay tuned for Part 1, where I work on the clock generator.  I plan on Part 2 being the receive mixer, but that will be a couple of weeks away.