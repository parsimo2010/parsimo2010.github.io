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
OscarSDR is designed to work up to, and in, the 2-meter amateur radio band (a "ham band").  In much of the world, the 2-meter ham band covers 144 to 148 MHz.  It is called the 2-meter band because radio frequencies that are about 150 MHz have a wavelength of about 2 meters.  Saying "2-meters" is much easier than saying "144-148 MHz band."  

Many people already know this, but radio waves in the 2-meter band can't be seen, heard, or sensed by humans.  So how does a radio receiver turn that wave into something useful, like audible sound waves, or data readable by a computer?  The first step is to bring the frequency down to something manageable for the rest of the radio.  Most radios that transmit and receive in the 2-meter band do most of the "complicated" work at a lower intermediate frequency (IF), such as just a couple of megahertz, or at baseband, which is just a few kilohertz for audio signals.  It is easier and cheaper to design circuits that work at these lower frequencies, and then convert the final signal to the desired radio frequency (RF) than it is to design a circuit that does everything at RF with no conversion.  These radios that do everything at RF do exist, they are called direct sampling radios and are typically very expensive.  OscarSDR aims to be cheap and made with easily available parts, so we have to design a stage that converts received signals down to a lower IF.  The way we accomplish this is simple mathematically, but requires a fairly complicated circuit to work well in the real world.

## The math of frequency conversion
Frequency conversion is done by multiplying the waves together.  Typically radio waves are described mathematically as a sine wave with frequency $\omega$.  There is a trigonometric identity that says:

$$sin(\omega_1 t) sin(\omega_2 t) = \frac{1}{2} (cos((\omega_1 + \omega_2) t) + cos((\omega_1 - \omega_2) t).$$

All this means is that if you 

![The top of the bad board](/assets/images/BadBoard1.png)
