Here are my steps for building and installing Firmware from scratch on a Windows 10 machine.

If you don't want to build a firmware, you just want to install one, you can download an [official firmware from Github](https://github.com/pingdynasty/OpenWare/releases) and then skip to step 12.

If you are planning to change the firmware code before building it, you may want to run the OPTIONAL, SCARY STEPS at the bottom of this guide first, for safety.

**Steps:**

1. I installed Ubuntu for Linux for Windows.

2. I downloaded and installed "64-bit java for windows". [https://www.java.com/en/download/windows-64bit.jsp](https://www.java.com/en/download/windows-64bit.jsp)

3. I rebooted.

4. I signed up for an account on [st.com](https://www.st.com).

5. I installed STM32CubeMX. [https://www.st.com/en/development-tools/stm32cubemx.html](https://www.st.com/en/development-tools/stm32cubemx.html)

6. I checked out [https://github.com/pingdynasty/OpenWare](https://github.com/pingdynasty/OpenWare) with Ubuntu for Linux git.

	- Let's call the directory you checked this out to `$OPENWARE_DIR`. You can save that to a shell variable with ```export OPENWARE_DIR=`pwd` ```.

7. I ran STM32CubeMX.

	- I selected an existing project and opened Magus/Magus.ioc. I migrated the project. I clicked "generate"

		- You will get a warning about "HAL timebase". You can ignore it.

8. In Ubuntu for Windows, I installed gcc (with `sudo apt install gcc-arm-none-eabi`)

9. I DID THESE STEPS, BUT **YOU SHOULDN'T**:

	* Edited checkout directory `common.mk` and replaced the GIT_REVISION line with:

          GIT_REVISION = $(shell hg log --template='{bookmarks} {gitnode|short}' -r .) $(CONFIG)

    	This step is a byproduct of something weird about my computer (I use Mercurial rather than Git).

   	* Edited Magus/cube-update.sh and replaced contents with

          #!/bin/bash
          hg revert --no-backup Inc/usbh_conf.h
          hg revert --no-backup Src/usb_device.c Src/usb_host.c Makefile
          rm -f Src/usbd_audio_if.c Inc/usbd_audio_if.h

10. From checkout directory ran (cd Magus && ./cube-update.sh)

11. Ran `make magus TOOLROOT=` in checkout directory.

12. Check out [https://github.com/pingdynasty/FirmwareSender](https://github.com/pingdynasty/FirmwareSender) with git.

	- Let's call the directory you checked this out to `$SENDER_DIR`. You can save that to a shell variable with ```export OPENWARE_DIR=`pwd` ```.

13. Open the Visual Studio solution in Builds/VisualStudio2015, click the menu at the top that says "Debug" and switch to "Release", select "Build Solution" in the Build menu.

14. Run:

        $SENDER_DIR/Builds/VisualStudio2015/Win32/Release/ConsoleApp/FirmwareSender.exe -in $OPENWARE_DIR/Magus/Build/Magus.bin -save $OPENWARE_DIR/Magus/Build/Magus.bin.syx

15. Go to the [Firmware uploader web page](https://pingdynasty.github.io/OwlWebControl/firmware.html) and follow the instructions there, feeding it the Magus.bin.syx file.

**OPTIONAL, SCARY STEPS: INSTALLING THE SAFETY BOOTLOADER**

On some OWL family devices, there is a button you can hold down to kick the device into bootloader mode. The Magus doesn't have a button like this. The only way to boot the Magus into bootloader is to send it the "reboot into bootloader" MIDI message. This means if your Magus firmware has a bug that causes it to boot at launch, before it gets to the point of receiving MIDI, you cannot get into the bootloader at all, and your device is now a brick.

The solution to this is to install the newer, "safe" bootloader. This uses something called a "watchdog" which means if the Magus firmware ever hangs for a sufficiently long period of time, it will automatically fall into the bootloader. However there are two problems with the safety bootloader:

- It is hard to install, and
- Once you install it, you must ONLY install firmwares that are compiled for use with the safety bootloader. You can't use official firmware anymore. You can only use firmware you compiled yourself.

As long as you use only officially released firmwares, you do not need the safety bootloader, because these have been tested. But if you are going to be modifying the firmware code, probably you should switch to the safety bootloader and compile your firmwares with watchdog support. Here's how.

(Aside: Do you know how to solder? If so, you might want to instead follow [these instructions]([these instructions by antisvin](https://community.rebeltech.org/t/connecting-programmer-to-magus/1433/1).)

1. You need to buy an ST programmer. You can get this for about $30 from ST, or if you search "st programmer" on alibaba or amazon you can get a very cheap knockoff.

	- By the way, once you have the ST programmer, you can just stop here. If you have a programmer you have the option of simply letting the Magus crash and brick itself, and then using the ST programmer to rescue it from brick state using the ST programmer.

2. Unscrew the four screws on the underside of the device and remove the flat panel.

3. You should see a little white board inside the unit. To get it out, you'll have to remove the "IO board", which is that other board in the way.

    - To do this, unscrew the IN, OUT and EXT plugs-- yes, the actual plugs, the little hexagonal protrusions on the outside. You can do this with an 8MM ratchet.

    - Remove the IO card by pulling up.

4. Now you need to remove the little white board. It holds on to its sockets pretty tightly, so try gripping it at the edges rocking it back and forth from top to bottom until it comes loose.

5. Now it's time to connect the white board to the ST programmer. [This diagram](https://github.com/pingdynasty/OpenWareLab/blob/master/OWL_Digital/Legacy/Owl-digital-pinout.pdf) shows the names of the pins on (it's not the same revision as is in the Magus, but the programmer pins don't change). You will need to connect **SWDIO**, **GND**, **SWCLK**, and **3.3V**. There are many pins so make absolutely sure you connected the right ones! 

6. In Ubuntu for Windows, install openocd (with `sudo apt install openocd`)

7. Turn on the watchdog ("IWDG") in the bootloader and the Magus firmware:

    - Edit `MidiBoot/Inc/hardware.h`, and uncomment `#define USE_IWDG`

    - Edit `Magus/Inc/hardware.h`, and add `#define USE_IWDG` to the end.

8. Open STM32CubeMX (if you don't have this: run steps 2-5 of the Magus instructions)

    - Select an existing project and open MidiBoot/MidiBoot.ioc. Select "Migrate", then "Generate".

8. `make midiboot TOOLROOT=`

8MM RACHET, pull up
rock back and forth

./Builds/VisualStudio2015/Win32/Release/ConsoleApp/FirmwareSender.exe -in ../../../Downloads/Magus.bin -save ../../../Downloads/Magus.bin.syx -flash 0x8329dec8

15. Run `./Builds/VisualStudio2015/Win32/Release/ConsoleApp/FirmwareSender.exe -l` to get the number of your Magus.
./Builds/VisualStudio2015/Win32/Release/ConsoleApp/FirmwareSender.exe -in ../../../Downloads/Magus.bin -out 1 -num 0x8329dec8
		- WARNING: When RTOS used strongly recommended to use HAL timebase instead of Systick
		(on step 7)