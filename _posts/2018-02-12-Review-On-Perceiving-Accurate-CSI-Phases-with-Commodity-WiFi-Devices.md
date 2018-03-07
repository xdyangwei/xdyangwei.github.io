---
layout: post
title: "Review on Perceiving Accurate CSI Phases with Commodity WiFi Devices"
---

## Introduction

After reading the paper 'Perceiving Accurate CSI Phases with Commodity WiFi Devices',I got some new knowledge of the way how to handle inaccurate CSI measurements which we can get from commodity wifi devices.

The Channel State Information (**CSI**) describes how a signal propagates from the transmitter to the receiver and represents the combined effect of, for example, scattering, fading, and power decay with distance.CSI has been recently used for various motion- or location-based applications.As a result, accurate CSI measurements are of great significance to tremendous applications.

However,because of the hardware imperfection,CSI phase measurements includes various errors besides true CSI phases,for example, Carrier Frequency Offset (CFO),Sampling frequency offset (SFO),Packet detection delay (PDD) and PLL Phase Offset (PPO).In this paper,the author Zhuo not only remove these errors mentioned above,but also identify the unknown non-negligible **non-linear** CSI phase errors and then remove them from CSI measurements.After that,we can get **accurate** CSI measurements.

## Known Errors

As we said above,because of the hardware imperfection,CSI phase measurements includes various errors besides true CSI phases.Fig.1 illustrates signal processing in 802.11 n.We can analyze the source which causes the errors through Fig.1.    

<center>![]({{ site.url }}/pictures/hardware.png)</center>  

<center>Fig.1</center>

**CFO** :  Carrier Frequency Offset (CFO) which is caused by the unsynchronized central frequencies of a transmission pair leads to a time-varying CSI phase offset across subcarriers.    

**SFO** : The sampling frequencies of the transmitter and the receiver exhibit an offset(SFO) due to non-synchronized clocks, which can cause the received signal after ADC a time shift with respect to the transmitted signal.Then the time shift causes phase rotation errors.    

**PDP** : Packet detection delay(PDP) stems from energy detection or correlation detection which occurs in digital processing after down conversion and ADC sampling.Packet detection introduces another time shift with respect to the transmitted signal , which leads to packet-varying phase rotation error.     

**PLL** : The phase-locked loop (PLL) is responsible for generating the center frequency for the transmitter and the receiver, starting at random initial phase. As a result, the CSI phase measurement at the receiver is corrupted by an additional phase offset.

From the above known error sources, the measured CSI phases are mainly distorted with various phase ration errors and/or phase offset errors.For a transmission pair, the phase measurement $$\phi$$(i, k) for sub-carrier k in band i can be expressed as     

![有帮助的截图]({{ site.url }}/pictures/csi.png) (1)

where k ranges from -28 to 28 ( index 0 is reserved for carrier frequency) in IEEE 802.11n for 20MHz band width, $$\theta$$ ( i, k) denotes the true phase, $$\delta_i$$ is the timing offset at the receiver, including time shift due to PDD and SFO, *f*$$_s$$ is the sub-carrier spacing between two adjacent sub-carriers (i.e. 312.5KHz), $$\beta_i$$ is the total phase offset, and Z is the additive white Gauss measurement noise.     

**Note that, except for Z, other reported phase errors are linear with sub-carrier indexes.**

## Unknown Errors

In order to verify the conclusion 'phase errors are linear with sub-carrier indexes', Zhuo conduct conduct an experiment in a typical indoor environment with the length and width of the room being 12 meters and 10 meters with Atheros AR9380 Nics. They arrange the transmitter and the receiver in strong line-of-sight (LOS) condition with distance of 0.5 meter, and make the transmitter to transmit with its first antenna, denoted as Nc1, with a fixed transmitting power of 5dBm and the receiver to receive with all of its three antennas denoted as Nr1, Nr2 and Nr3 respectively.    

<div style="align: center">![有帮助的截图]({{ site.url }}/pictures/1.png)</div>    

<center>Fig.2</center>

Figure 2 illustrates three groups of unwrapped CSI phase measurements for three consecutive packets, with each group containing CSI phases for 1 by 3 transmission pairs.According to (2), the ideal phases
on different sub-carriers should be almost linear with the sub-carrier indexes. **They observe, however, obvious non-linear distortions in all unwrapped phase measurements.**They repeat such experiment in an indoor gymnasium with length of 50 meter and width of 30 meter, and get similar results.

<div style="align: center"><img src="{{site.url}}/pictures/11.png"></div>    

<center>Fig.3</center>     

According to previous work, if the wireless channel is stable, the unwrapped phase differences of two consecutive packets for the same transmission pair are almost linear.







