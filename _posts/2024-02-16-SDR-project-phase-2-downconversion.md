---
layout: posts
title:  "SDR Project Phase 2: Downconversion"
date:   2024-02-16
categories: sdr_project_log
author_profile: true
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

## Mixing it down (and up)
OscarSDR is designed to work up to, and in, the 2-meter amateur radio band (a "ham band").  In much of the world, the 2-meter ham band covers 144 to 148 MHz.  It is called the 2-meter band because radio frequencies that are about 150 MHz have a wavelength of about 2 meters.  Saying "2 meters" is much easier than saying "144-148 MHz band."  

Many people already know this, but radio waves in the 2-meter band can't be seen, heard, or sensed by humans.  So how does a radio receiver turn that wave into something useful, like audible sound waves, or data readable by a computer?  The first step is to bring the frequency down to something manageable for the rest of the radio.  Most radios that transmit and receive in the 2-meter band do most of the "complicated" work at a lower intermediate frequency (IF), such as just a couple of megahertz, or at baseband, which is just a few kilohertz for audio signals.  It is easier and cheaper to design circuits that work at these lower frequencies, and then convert the final signal to the desired radio frequency (RF) than it is to design a circuit that does everything at RF with no conversion.  These radios that do everything at RF do exist, they are called direct sampling radios and are typically very expensive.  OscarSDR aims to be cheap and made with easily available parts, so we have to design a stage that converts received signals down to a lower IF.  The way we accomplish this is simple mathematically but requires a fairly complicated circuit to work well in the real world.

## The math of frequency conversion
Frequency conversion is done by multiplying the waves together.  Typically radio waves are described mathematically as a sine wave with frequency \( \omega. \)  There is a trigonometric identity that says:

\[ sin(\omega_1 t) sin(\omega_2 t) = \frac{1}{2} (cos((\omega_1 + \omega_2) t) + cos((\omega_1 - \omega_2) t). \]

All this means is that if you multiply two perfect sine waves the result is two cosine waves with frequencies equal to the sum of the original frequencies and their difference.  Luckily, a cosine wave is identical to a sine wave with a 90-degree phase shift, so we can just treat these cosine waves the same as we would treat sine waves.  So multiplying two waves (the RF and output from the VFO from part 1) is useful for making their frequencies both higher *and* lower.  In the radio world, we call multiplying two frequencies "mixing," and we use filtering to eliminate the frequency that we don't want.

## It's never that easy in real life
Unfortunately in real life, we never have two pure sine waves, and they aren't the same amplitude, and our circuits don't behave like perfect math operators.  A good mixer will do a lot to help us out, they typically contain amplifiers and other signal conditioning to use whatever inputs are provided to the mixer.  But these operations are inherently non-linear, and we end up with undesired outputs that we have to deal with.  Luckily these undesired outputs (also called intermodulation products) are fairly well understood, and we have a way to deal with them in our radio design.

## Testing our mixer
I picked the Analog Devices AD831 RF mixer to use in OscarSDR because it is affordable and widely available.  It works for frequencies up to 500 MHz so it can handle the 2-meter band, amplifies the inputs to get the signals to useable levels, and can be used with single-ended power supplies (positive voltage only) so there is no need to generate negative voltages.  The one downside is that it wants between 9 and 11 volts of input for its power supply, so we cannot power this from the 3.3V or 5V available from a Pi Pico.  Wall power supplies that output 9V are cheap, so this is not too big of a deal.  

We need to test our mixer on known inputs, so we will feed the outputs of the SI5351 into both inputs of the mixer to check for our expected outputs.  Below is an image of the test setup, which consists of my computer writing programs for the Pi Pico, which is used to configure the SI5351 clock generator to produce outputs of 3 and 5 MHz, which are fed into the green AD831 board, and measured with the oscilloscope.  Below that picture are images of the oscilloscope screen once everything was working.

![The test setup](/assets/images/phase2/scope.jpg)
![The result in the time domain](/assets/images/phase2/results1.bmp)
![The result in the frequency domain](/assets/images/phase2/results2.bmp)

## What's all that?
An astute reader would notice that the output on the oscilloscope looks nothing like two sine waves.  There are two reasons- the first is that the 3 MHz and 5 MHz clock signals from the SI5351 are square waves.  The second reason is the intermodulation products mentioned above.  Typically once we get to complicated signals that are not a single frequency, we can better understand them by looking at the signal in the frequency domain instead of the time domain.  My cheap oscilloscope has math functions and includes a Fast Fourier Transform (FFT), which is the purple graph in the images below.  In the frequency domain, you can see the expected sum and difference frequencies of 2 MHz (from 5-3) and 8 MHz (from 5+3), but you also see intermodulation products every 2 MHz.  This is fine, and since we know about this behavior we can make sure we have a plan to address it (in future parts).  However, proving that the mixer works with strong clock signals doesn't mean that it will work for what we want to do, which is receiving RF signals from the open air.  So we want to do one more test before proceeding to the next stages of building OscarSDR.

## The actual test begins now
So now we have a functioning mixer, let's dig a radio out and see if we can do something with real, honest-to-goodness electromagnetic waves through the air instead of signals being conducted by copper.  I have a Baofeng UV-5R just like every other amateur radio operator.  Baofengs are like mopeds- everybody loves them but nobody wants to admit it.  Anyway, the pictures below show the modified test setup.  The Baofeng is set up to transmit at 145.20 MHz on its lowest power setting.  I have a stub antenna attached to the RF input of the mixer, and the SI5351 is set to output a clock signal at 144.2 MHz, which is 1 MHz below the RF.  So when the Baofeng transmits we should expect an IF of 1 MHz from the difference frequency.  We also expect to see the sum frequency of 289.4 MHz appearing as noise, and a bunch of other intermodulation products.  And that's exactly what we got in the second image below.  Success!

![The test setup with a radio](/assets/images/phase2/scope_w_radio.jpg)
![The super gnarly result](/assets/images/phase2/results3.bmp)

## Steps for higher performance
A more expensive and higher performing radio would put a clock signal through a low pass filter before feeding into the mixer, as well as putting the input from the antenna through a band pass antenna.  These actions would cut down on the undesired products coming out of the mixer and deliver a higher signal-to-noise ratio to later stages in the receiver.  I want to keep OscarSDR cheap and simple, so I will skip the filtering until it becomes necessary (such as for the transmit stage).  I reserve the right to change my mind and add filtering later, but for now, I am going to accept the lower performance.