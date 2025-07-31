---
date: "2025-07-30T12:31:35-04:00"
draft: true
title: Programming the Netduino with Arduino
description: "Believe it or not, it's an Arduino compatible device."
tags: ["Netduino", "Arduino", "STCube"]
categories: ["Arduino", "MCU"]
series: ["Netduino"]
ShowToc: true
---

If you have one of these in your toolbox, or have ever had one, you might've thought they were obsolete.  Their support and tools are still around but unfortunately have gone dormant and the toolset may not even work on modern operating systems.  However, they aren't useless and still can do some pretty amazing things.  Interested?  Keep reading.  This post will explore how to setup the Arduino IDE to program your Netduino 2.  

Yep, that's possible and without crossing wires, waving any magic wands, or opening a worm hole to an alternate universe.

![Netduino 2](/Netduino2.jpg)

Note:  The details here are specifically for Netduino 2, but there is a strong possibility that they will work with other Netduino boards.  If you own a different Netduino based on the STM32 chipset, please try these steps and let me know if they work.

# Board Specifics

The Netduino 2 board is built with a standard Arduino Uno compatable footprint.  It has an external 25Mhz clock crystal, [USB OTG](https://en.wikipedia.org/wiki/USB_On-The-Go) support, power LED, user-programmable blue LED, user/boot button, 6 Analog pins, and 13-15 digital pins.  If you take a look at the [Schematic](/Schematic_N2_20Dec16.pdf), you'll find the microcontroller is the [STM23F205RFT6](https://www.st.com/en/microcontrollers-microprocessors/stm32f205rf.html), a high-performance ARM&reg; Cortex&reg;-M2 32-bit processor manufactured by STMicroelectronics with a [10 year longevity committment](https://www.st.com/content/st_com/en/support/resources/product-longevity.html#10-year-longevity).  Which means they'll support it until at least 2035.  If you're interested you can read more about this chip in it's [datasheet](https://www.st.com/resource/en/reference_manual/rm0033-stm32f205xx-stm32f207xx-stm32f215xx-and-stm32f217xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf). 

When [Wilderness Labs LLC](https://www.wildernesslabs.co) designed this board they kept pretty close to the reference design, which means you're able to use all of the tools in the [STMCube ecosystem](https://www.st.com/content/st_com/en/ecosystems/stm32cube-ecosystem.html) to program this device (more details on that in future posts).  But, whats even more intriguing is that you can use the [STM32duino](https://github.com/stm32duino) board support package to enable the Arduino IDE to recognize, program and possibly even debug this board.  Wow! I have to admit I really didn't think that was possible.  Before I tried this, I thought this board would only handle development using .Net MF development.  Well hold on, we're going to dive right in.

# ST Arduino Support



