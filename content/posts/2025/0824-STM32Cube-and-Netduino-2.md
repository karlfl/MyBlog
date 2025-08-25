---
date: "2025-08-23T20:35:35-04:00"
draft: true
title: STM32Cube and the Netduino 2
description: "Using the STM32Cube ecosystem to program the Netduino 2"
tags: ["Netduino", "Arduino", "STCube", "STM32"]
categories: ["Arduino", "MCU", "STM32"]
series: ["Netduino"]
ShowToc: true
---

The Netduino 2 was a somewhat popular development board that implemented the familiar Arduino board layout and used the .Net Microframework for programming it.  Unfortunately these units have been discontinued and their .Net development tools have become abandoned.  However, I suspect that some of you may be like me and still have one in a closets or box collecting dust.  Please **don't throw them away**.  They are still useful boards and if you're willing to take a few minutes to setup a few other tools you'll see for yourself.  Don't let the name fool you, these devices are not exclusive to the .Net framework and have many different personas.  The board contains a powerful 32-bit ARM MCU (micro controller unit) with a 25Mhz external clock crystal.  The MCU is a variant of the STM32F205 which has a lot of capabilities.  The number of peripherals this chip supports is quite impressive.  The Netduino 2 board boasts 22 GPIO pins, 6 PWM Channels, 4 UART ports, I2C, SPI and more.  If you [explore the schematic](/Schematic_N2_20Dec16.pdf), you'll see it has several useful features including a USB-OTG interface that allows programming via the built in STM32 DFU (Device Firmware Update) mode.

![Netduino 2](/Netduino2.jpg)

[A few weeks ago]({{< ref "posts/Reviving-My-Netunio-2">}}) I wrote about my journey of reviving a couple of Netduino 2 boards that had gotten lost in the back of my electronics box.  That article included a basic set of instructions for programming the Netduino with both the Arduino IDE and the STM32Cube tools.  [My second article]({{< ref "posts/Programming-Netduino-in-Arduino">}}) took a closer look at the Arduino IDE giving more detailed configuration and programming steps for using this familiar development environment.  I'd definitely recommend the Arduino IDE for programming these boards.  The amount of examples and assistance available for Arduino makes using it a breeze and since this board is compatable with most 

This post will focus on the more advanced STM32Cube toolset.  Here we'll learn how to setup and configure the board and write some C code to show off the boards basic features.  We'll still keep it pretty simple so even those that haven't used STM32Cube before should be able to follow along without too many issues.

# Software Setup

To begin with you'll need to make sure you have a few products installed on your PC.  If you've followed along with the other articles you may have a few of these already installed.  Since I use VS Code for a lot of other projects, I've chosen to use [STM32Cube for VS Code](https://www.st.com/content/st_com/en/stm32-mcu-developer-zone/software-development-tools/stm32cubevscode.html).  But if you're more familiar with the Eclipse IDE feel free to download and install the full [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html).

## Prerequisites

* [STM32CubeCLT](https://www.st.com/en/development-tools/stm32cubeclt.html)
* [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html)
* [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)
* [STM32Cube for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=stmicroelectronics.stm32-vscode-extension&ssr=false#overview)

## Configuration
Once you've installed the tools above you'll need to tell the VS Code extension where to find the CubeMX and Programmer.  Go back to the VS Code extension and use the gear icon to get to the settings window.

![STM32Cube for VS Code Settings](/STM32CubeforVSCodeSettings.png)

Here's a snapshot of my settings page.  If you took the defaults during installation yours should be in similar locations.

![STM32Cube for VS Code Path Settings](/STM32CubeforVSCodePathSettings.png)


# Creating Our First Project

It's time to get that LED blinking.  You should have a new icon in the activity bar on the left side that should take you to the STM32 VS Code Extension tools window.  Let's start with the Launch STM32CubeMX link.  As it states this opens the STM32CubeMX tool that will get us started creating our code project.  You may notice that it will download some packages.  If you get any errors while opening this tool, make sure you've created an account on the STM32 site.  I found that once I'd logged into their site I no longer saw that error.  Once the window opens select the 'Start My project from MCU' link in the middle of the screen.  Once the product selector window opens, make sure the "MCU/MPU Selector" tab is selected and enter 'STM32F205RFT6' (that's the exact MCU used in the Netduino 2) in the Commercial Part Number box, then highlight that same part number in the list on the bottom right.  Once selected the top right box will display details about the MCU, including links to ST product pages and datasheet.  Useful if you want to dive deeper into this product line.  You're screen should look something like this.

![STM32CubeMX Product Selector](/STM32CubeMXProductSelector.png)

Now press teh "Start Project button in the top right which will gather the initial project details and download any MCU specific files necessary for code generation.  Once complete it will take you back to the STM32CubeMX window with the Pinout & Configuration page opened.

![STM32CubeMX Pinout and Configuration](/STM32CubeMXPinoutAndConfiguration.png)

# Schematics and Pin Configuration

We'll take a short pause here to talk a little bit about the Netduino 2 design and specifically how the board is wired.   This information is important for filling out the pinout details and configuring the clock with the STM32CubeMX tool.  You'll find a copy of the Netduino 2 Schematic [here on my blog](/Schematic_N2_20Dec16.pdf), or within the [Netduino Github repo](https://github.com/WildernessLabs/Netduino_Hardware/blob/main/N2/N2_20Dec16.pdf).  When you have time I'd suggest that you review this document in detail.  It will reveal to you a lot of information about how the MCU is wired up on the board and assist you when you want to configure your STM32Cube project to use a specific pin or peripheral.

*Keep in mind I'm not an electrical engineer, so a lot of this is quite foreign to me. You should take any information in this section with some caution as it may not be 100% correct.*

For now I want you to locate the external crystal configuration found just below the green Microcontroller text and to left of the MCU block.  This picture should help you find it.  You'll find it wired to PH0/PH1 (pins 5/6).  This is the 25Mhz crystal oscillator used as the clock source for the STM32F205 MCU.  While, it's not critical to understand all the details about how this is wired, you'll find it helpful when we go back to the pinout and configuration section of the STM32CubeMX tool.  

![Netduino 2 Crystal and USB Schematic](/Netduino2SchematicUSBConnectors.png)

You should also locate the USB Micro-B connector (on the top left in the picture above) and follow the 5V, D- and D+ lines over to the MCU.  They will be mapped to OTC_FS_VBUS, OTC_FS_DM and OTG_FS_DP or PA9, PA11 and PA12 (pins 42/44/45) respectively.  While you're in this same section you should take note of the LED wired to PA10 (pin 43).  This is the blue user LED found on the Netduino 2.  It has a pull-up resistor wired to +3.3V (*this means to turn it on we'll need to set the pin LOW or 0, which may seem a little odd when programming it*). Ok, now we have enough information to begin our basic pin and clock configurations.  We'll come back to the schematic later when we explore the user BTN and a few other cool wiring pieces the engineers at Secret Labs LLC (now Wilderness Labs LLC) built into this board.  Let's jump back to the STM32CubeMX window.

# Clock Configuration

Back in STM32CubeMX, expand the **System Core** section under the 'Categories' tree on the left side.  Click on **RCC** (Realtime Clock Configuration).  In the **Mode** tool window that comes up, set the **High Speed Clock (HSE)** to '**Crystal/Ceramic Resonator**'.  You notice that the pin diagram on the right will have changed and you'll see that pins PH0/PH1 are now green with RCC_OSC_IN and RCC_OSC_OUT next to them.  Now the schematic diagram notes we made above should make a lot more sense.  

![STM32CubeMX External Clock Pins](/STM32CubeMXExternalClockPins.png)

We'll need make a few more adjustments before we finish the clock configuration.  All we've done so far is tell the tool that our board is using an external high speed clock.  We haven't told it anything about the frequency of the clock.  Let's do that now by going to the **Clock Configuration** tab.  This tab may seem overwhelming, it did for me when I first looked at it.  I had no idea what any of this was.  Don't worry the tool has some magic built in that will take care of most of this for us.  For now we'll only need to make a few minor changes to the settings on this page and then we'll tell SMT32CubeMX calculate the remainder of the settings.  

First double check that the Input Frequency box on the left is set to 25 Mhz.  Next, follow the HSE line to the right and select the **"HSE"** radio button below in the **PLL Source Mux** box.  Continue following the arrows to the right and select the **PLLCLK** radio button in the **System Clock Mux** box.  You may see a few boxes go red.  That's to be expected and we'll correct that in a minute.  Your part of the clock configuration is done.  Now we'll let the tool to do it's magic.  In the top toolbar clock the **Resolve Clock Issues** button.  This should modify several of the boxes and you shouldn't see any more red boxes.  Here's how your screen should look now.

![STM32CubeMX Clock Configuration](/STM32CubeMXClockConfiguration.png)

# USB and LED Pin Setup

We're almost there.  Only a few more things to configure and we'll be ready to generate our code.  Head back to the **Pinout & Configuration** tab.  We'll start with the USB configuration.  Expand the **Connectivity** section in the tree view on the left and select USB_OTC_FS.  in the **Mode** tool window that comes up, choose **'Device Only'** from the **Mode** dropdown and check the **Activate_VBUS** checkbox.  You should see the PA9/11/12 go green with 'USB_OTG_FS_xx' labels next to them. 

*The USB settings might not be necessary for our simple blink program, but I find it helpful to configure it now so I can easily see what pins are already allocated to a specific peripheral.*

Let's jump back to the **Clock Configuration** tab because our USB settings may have caused an error back there.  You may have noticed a red x on this tab and when you see the page you'll probably notice a few additional boxes are now also red.  That's because there is another clock configuration piece needed now that we have the USB_OTG_FS configured.  The simple fix for this is to just let it **Resolve Clock Issues**.  Goodbye red boxes.

![STM32CubeMX USB Configuration]()

Last configuration step.  Let's get that LED wired up.  Back to the **Pinout & Configuration** tab.  Using the pin diagram in the right window, locate the **PA10 pin** on the top right corner between the USB_OTG_FS pins.  If you remember our schematic notes, this is where the blue LED is wired to.  Place you cursor on that pin and **left click**.  A dropdown menu will appear showing all the peripheral options available on that pin.  Select **GPIO Output**.  Back in the left tree view select **System Core** --> **GPIO**.  In the bottom **Configuration** tool window click on the **PA10** entry and set the **User Label** to **'LED'**.  Setting this label isn't critical, but it'll make our code a little easier to read since we can refer to the pin by it's label.



That's it.  Time to generate some code and get that LED blinking.

