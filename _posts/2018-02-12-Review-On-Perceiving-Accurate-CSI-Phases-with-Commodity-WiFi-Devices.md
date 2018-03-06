---
layout: post
title: "Review on Perceiving Accurate CSI Phases with Commodity WiFi Devices"
---

## Introduction

After reading the paper 'Perceiving Accurate CSI Phases with Commodity WiFi Devices',I got some new knowledge of the way how to handle inaccurate CSI measurements which we can get from commodity wifi devices.

The Channel State Information (**CSI**) describes how a signal propagates from the transmitter to the receiver and represents the combined effect of, for example, scattering, fading, and power decay with distance.CSI has been recently used for various motion- or location-based applications.As a result, accurate CSI measurements are of great significance to tremendous applications.

However,beacause of the hardware imperfection,CSI phase  measurements includes various errors besides true CSI phases,for example, Carrier Frequency Offset (CFO),Sampling frequency offset (SFO),Packet detection delay (PDD) and PLL Phase Offset (PPO).

CFO which is caused by the unsynchronized central frequencies of a transmission pair leads to a time-varying CSI phase offset across subcarriers.The sampling frequencies of the transmitter and the receiver exhibit an offset(SFO) due to non-synchronized clocks, which can cause the received signal after ADC a time shift with respect to the transmitted signal.Then the time shift causes phase rotation errors.Packet detection delay(PDP) stems from energy detection or correlation detection which occurs in digital processing after down conversion and ADC sampling.Packet detection introduces another time shift with respect to the transmitted signal , which leads to packet-varying phase rotation error.The phase-locked loop (PLL) is responsible for generating the center frequency for the transmitter and the receiver, starting at random initial phase. As a result, the CSI phase measurement at the receiver is corrupted by an additional phase offset.

From the above known error sources, the measured CSI phases are mainly distorted with various phase ration errors and/or phase offset errors.For a transmission pair, the phase measurement $$\phi$$(i, k) for sub-carrier k in band i can be expressed as ![csi]({{site.url}}/pictures/csi.png)







