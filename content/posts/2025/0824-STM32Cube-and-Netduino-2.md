---
date: "2025-08-23T20:35:35-04:00"
draft: false
title: STM32Cube and the Netduino 2
description: "Using the STM32Cube ecosystem to program the Netduino 2"
tags: ["Netduino", "Arduino", "STM32Cube", "STM32"]
categories: ["Arduino", "MCU", "STM32"]
series: ["Netduino"]
ShowToc: true
---

Do you own one of these?  I suspect that some of you may be like me and still have one in a closet or box collecting dust.  Please, **don't throw them away**.  They are still useful boards and if you're willing to take a few minutes to setup a few tools you'll see for yourself.  Don't let the name fool you, these devices are not exclusive to the .Net framework.  The board contains a powerful 32-bit ARM MCU (micro controller unit) with a 25Mhz clock crystal pushing the internal speed of some components to 120Mhz.  The MCU is a variant of the STM32F205 which has a lot of capabilities.  The number of peripherals this chip supports is quite impressive.  The Netduino 2 board boasts 22 GPIO pins, 6 PWM Channels, 4 UART ports, I2C, SPI and more.
![Netduino 2](/Netduino2.jpg)

[A few weeks ago]({{< ref "posts/Reviving-My-Netunio-2">}}) I wrote about my journey of reviving a couple of Netduino 2 boards that had gotten lost in the back of my electronics box.  That article included a basic set of instructions for programming it with both the Arduino IDE and the STM32Cube tools.  [My second article]({{< ref "posts/Programming-Netduino-in-Arduino">}}) took a closer look using the Arduino IDE for programming a ~~Net~~duino programming.  I'd definitely recommend that setup for programming these boards.  The amount of examples and assistance available for Arduino makes using it a breeze and this board is compatible with a lot of the shields available for Arduino.

This post will focus on the more advanced STM32Cube toolset.  Here we'll learn how to setup and configure the board and write some C code to show off the boards basic features.  We'll still keep it pretty simple so even if you haven't used STM32Cube before, it should be easy to follow along.

# Software Setup

To start, you'll need to make sure you have a few tools installed on your PC.  If you've followed along with the other articles you may have a few of these already installed.  Since I use VS Code for a lot of other projects, I chose to use the [STM32Cube for VS Code](https://www.st.com/content/st_com/en/stm32-mcu-developer-zone/software-development-tools/stm32cubevscode.html) extension.  [STMicroelectronics has a short video](https://youtu.be/DLmbNfUh62E) walking through the install and configuration.

## Prerequisites

* [STM32CubeCLT](https://www.st.com/en/development-tools/stm32cubeclt.html)
* [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html)
* [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (I installed this but it may be optional)
* [STM32Cube for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=stmicroelectronics.stm32-vscode-extension&ssr=false#overview)

## Configuration

Once you've installed the tools above you'll need to tell the VS Code extension where to find the CubeMX and CLT.  Go back to the VS Code extension and use the gear icon to get to the settings window.

![STM32Cube for VS Code Settings](/STM32CubeforVSCodeSettings.png)

Here's a snapshot of my settings page (Linux).

![STM32Cube for VS Code Path Settings](/STM32CubeforVSCodePathSettings.png)


# Creating Our First Project

It's time to get that LED blinking.  Bring up the VS Code window if it's not already up.  You should have a new icon in the activity bar on the left that should reveal the STM32Cube tools window.  Let's start with the Launch STM32CubeMX link.  As the external tool opens, you may notice it download some support packages.  If you get any errors during this step, create an account on the STM32 site.  I found that once I created my account and logged into their site I no longer saw that error.  Once the tool opens, select the 'Start My Project from MCU' link in the middle of the screen.  This will open the product selector window open. Make sure the "MCU/MPU Selector" tab is selected and enter **STM32F205RFT6** in the **Commercial Part Number** box.  Now highlight that same part number in the list on the bottom right.  Once selected, the top right section will display details about the MCU, including links to ST product pages and datasheet.  Useful if you want to dive deeper into this product line.  You're screen should look something like this.

![STM32CubeMX Product Selector](/STM32CubeMXProductSelector.png)

Now press the **Start Project** button in the top right which will setup the initial project details and download any MCU specific files necessary for code generation.  Once complete it will take you back to the STM32CubeMX window with the Pinout & Configuration page opened.

![STM32CubeMX Pinout and Configuration](/STM32CubeMXPinoutAndConfiguration.png)

# Schematics and Pin Configurations

I'm going to take a short pause here to talk more about the Netduino 2 design, specifically how the board is wired.   This information is important for filling out the pinout details and configuring the clock with the STM32CubeMX tool which we just opened.  You'll find a copy of the Netduino 2 Schematic [here on this site](/Schematic_N2_20Dec16.pdf), or within the [Netduino Github repo](https://github.com/WildernessLabs/Netduino_Hardware/blob/main/N2/N2_20Dec16.pdf).  When you have time I'd suggest that you review this document in detail.  It will reveal to you a lot of information about how the MCU is wired up on the board and assist you when you want to configure your STM32Cube project to use a specific pin or peripheral.

*Keep in mind I'm not an electrical engineer, so a lot of this is foreign to me, but this is how we learn.  Exploring things that may not make sense to us now, can reap great benefits in the future.  Cautious curiosity is a great skill to master.*

For now I want you to locate the external crystal configuration found just below the green Microcontroller text and to left of the MCU block.  This picture should help you find it.  It's wired to PH0/PH1 (pins 5/6) on the microcontroller.  This is the 25Mhz crystal oscillator used as the clock source for the STM32F205 MCU.  (*I suspect that the oscillating frequency shown in the schematic, 25000Mhz, is a typo, since I think that translates to 25Ghz and I'm told this board runs at 25Mhz or 25000Khz.*)  It's not critical to understand all the details about how this is wired, however you'll find it helpful when we go back to the STM32CubeMX tool.

![Netduino 2 Crystal and USB Schematic](/Netduino2SchematicUSBConnectors.png)

Next, locate the USB Micro-B connector (top left in the picture above) and follow the 5V, D- and D+ lines over to the MCU.  They will be mapped to OTC_FS_VBUS, OTC_FS_DM and OTG_FS_DP or PA9, PA11 and PA12 (pins 42/44/45) respectively.  While you're in this same section you should take note of the LED wired to PA10 (pin 43).  This is the blue user LED found on the Netduino 2.  It has a pull-up resistor wired to +3.3V (*this means to turn it on we'll need to set the pin LOW or 0, which may seem a little odd when programming it* ). Ok, now we have enough information to begin our basic pin and clock configurations.  We'll come back to the schematic later when we explore the user BTN and a few other cool wiring pieces the engineers at Secret Labs (now Wilderness Labs) built into this board.  Let's jump back to the STM32CubeMX window.

# Clock Configuration

Expand the **System Core** section under the **Categories** tab on the left side.  Click on **RCC** (Reset and Clock Control).  In the **Mode** window that comes up, set the **High Speed Clock (HSE)** to '**Crystal/Ceramic Resonator**'.  Notice the pin diagram on the right has changed. The pins PH0/PH1 are now green with RCC_OSC_IN and RCC_OSC_OUT next to them.  This matches what we saw on the schematic diagram, pretty cool.

![STM32CubeMX External Clock Pins](/STM32CubeMXExternalClockPins.png)

We've configured our system to use an external high speed clock crystal, but we haven't told it anything about the clock or how we want to use it.  Let's do that now by going to the **Clock Configuration** tab.  This tab may seem overwhelming, it did for me when I first looked at it. Don't worry the tool has some magic built in that will take care of most of this for us.  For now we'll only need to make a few changes on this page and then we'll tell SMT32CubeMX figure out the rest.  

First double check that the HSE Input Frequency box on the left is set to 25 Mhz.  Next, follow the HSE line to the right and select the **HSE** radio button below in the **PLL Source Mux** box.  Continue following the arrows to the right and select the **PLLCLK** radio button in the **System Clock Mux** box.  You may see a few boxes go red.  That's to be expected and we'll correct that in a minute.  Your part of the clock configuration is done.  Now we'll let the tool to do it's magic.  In the top toolbar click the **Resolve Clock Issues** button.  This should modify several of the boxes which should remove those red boxes.  Here's how your screen should look now.

![STM32CubeMX Clock Configuration](/STM32CubeMXClockConfiguration.png)

# USB and LED Pin Setup

We're almost there.  Only a few more things to configure and we can generate our code.  Head back to the **Pinout & Configuration** tab.  We'll start with the USB configuration.  Expand the **Connectivity** section in the tree view on the left and select USB_OTC_FS.  In the **Mode** tool window that comes up, choose **Device Only** from the **Mode** dropdown and check the **Activate_VBUS** checkbox.  You should see the PA9/11/12 go green with 'USB_OTG_FS_xx' labels next to them. 

*The USB settings might not be necessary for our simple blink program, but I find it helpful to configure it now so I can easily see what pins are already allocated for a dedicated purpose.*

Let's jump back to the **Clock Configuration** tab because our USB settings may have caused an error back there.  You may have noticed a red x on this tab and when you see the page you'll probably find a few other boxes are now red.  That's because there is another clock configuration piece needed now for the USB_OTG_FS configured.  The simple fix for this is to use the **Resolve Clock Issues** button one more time.  Goodbye red boxes.

<!-- ![STM32CubeMX USB Configuration](/STM32CubeMXUSBClockConfig.png) -->

Last configuration step.  Back to the **Pinout & Configuration** tab.  Let's get that LED setup.  Using the pin diagram in the right window, locate the **PA10 pin** on the top right corner between the USB_OTG_FS pins.  If you remember our schematic notes, this is where the blue LED is wired to.  Place your cursor on that pin and **left click**.  A dropdown menu will appear showing all the peripheral options available on that pin.  Select **GPIO Output**.  Back in the left tree view select **System Core** --> **GPIO**.  Go to the **Configuration** tool window and click on the **PA10** entry and set the **User Label** to **LED**.  Setting this label isn't critical, but it'll make our code a little easier to read since the code can refer to the pin by it's label.  Here's a final screenshot of the CubeMX pinout view with the LED pin settings.

![STM33CubeMX LED Pin Configuration](/STM32CubeMXLEDOutput.png)
<!-- <img src="/STM32CubeMXLEDOutput.png" alt="STM33CubeMX LED Pin Configuration" width="500" height="300"> -->

That's it.  Time to generate some code and get that LED blinking.   

# Project Manager and Generating Code

Go to the **Project Manager** tab.  Fill in the **Project Name**, **Project Location**, and **Toolchain/IDE** (I used CMAKE & GCC since I'm on Linux and using VS Code).  The rest you can leave at their defaults.  Now click the **Generate Code** button above the Tools tab.  This may perform a download of the MCU specific code framework.  When it's complete it'll give you a dialog box.  Take note of the location that the code was generated to.  We'll need this in a minute.  

You can leave STM32CubeMX open and go back to (or re-open) VS Code.  Click the STM32 button in the activity bar to display the STM32 tools window.  Then click **Import CMake Project** and locate the project folder listed on that dialog box when you generated your code. Click **Open**.  Confirm that the Import settings match what you selected. and click the **Import Project** action, then select **Open in this window** button.  If VS Code prompts you for a preset choose Debug.

Whew, lots of things just happened.  Thankfully we don't have to understand it all at this point.  Remember our goal is to get that LED blinking.  To do that we need to add some code to the *main()* loop.  Open **/Core/Src/main.c** file.  Scroll down to the **while(1)** loop around line 98 and add this code just after the *USER CODE BEGIN 3* comment and before the closing *}*.  Your loop should look like this.

```C
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
      // Turn LED on (remember setting the pin LOW (0) turns on the LED)
      HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_RESET);
      HAL_Delay(500); // pause for .5 sec

      // Turn LED off
      HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_SET);
      HAL_Delay(500); // pause for .5 sec

  }
  /* USER CODE END 3 */

```

```C
      // OPTION B - Toggle the pin between HIGH (1) and LOW (0)
      HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
      HAL_Delay(1000); // pause for 1 sec
```

That's all the code needed.  Right click on the CMakeLists.txt file in the Explorer window and select **Build All Projects**.  When it's complete you should see something like this in the *OUTPUT* window.

```
...
[build] [23/23] Linking C executable Netduino_LED2.elf
[build] Memory region         Used Size  Region Size  %age Used
[build]              RAM:        3848 B       128 KB      2.94%
[build]            FLASH:        8032 B       768 KB      1.02%
[driver] Build completed: 00:00:02.862
[build] Build finished with exit code 0
```



# Other Schematic observations.

   - Power LED is Switchable
   - BTN is wired to two GPIO ports (PC14/PB11)
   - Power header (3v & 5V) can be turned on and off