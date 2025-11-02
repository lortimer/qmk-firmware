# QMK Firmware Setup for Ploopy Adept

Following https://github.com/ploopyco/adept-trackball/wiki/Appendix-D%3A-QMK-Firmware-Programming to be able to customize my own firmware for the ploopy adept.

## Setup QMK

Following https://docs.qmk.fm/newbs_getting_started

I decided to put the qmk firmware home directory in this folder, rather than the default location of ~/qmk-firmware. I chose this so it wouldn't pollute my home folder, and would be contained in this project for easy viewing of files in my IDE. I did this by running `qmk setup -H ./qmk-firmware-home`

Installed QMK in virtualenv in this project. When I ran `qmk setup`, it first said it couldn't find the qmk firmware, and would I like to clone it to the qmk home folder? I said yes and it succeeded. 

It then asked if I wanted to set ./qmk-firmware-home as QMK's home folder and I said yes. 

It then found a bunch of missing and outdated udev rules for specific keyboards. It told me to copy a udev rules file to /etc/udev/rules.d. It didn't let me do it at that point, but I did it later.

Then it couldn't find a bunch of dependencies: arm-none-eabi-gcc, avr-gcc, avrdude, dfu-programmer, and dfu-util and asked if I wanted to install them. I said yes.

Setup succeeded, and after I copied the udev rule, it succeeded without warnings.

## Testing QMK by compiling default firmware

I ran `qmk compile -kb ploopyco/madromys/rev1_001 -km default`, which compiled the default firmware into a uf2 file in the qmk home folder.

## Creating a keymap

Didn't do this, see Vial instructions below

## Flashing Vial-compatible firmware

I don't need to make my own keymap. I want to use Vial, which is much more feature-rich than Via. To do that, all I have to do is flash firmware compatible with Vial onto the trackball. To do this, I followed this from Vial: https://get.vial.today/manual/first-use.html#4-my-keyboard-isnt-discovered-or-it-doesnt-work, and this from QMK:https://docs.qmk.fm/newbs_flashing

Putting the Adept into bootloader mode involves plugging it in while holding the bottom-left mouse button.

I ran `qmk flash -kb ploopyco/madromys/rev1_001 -km vial`.I got an error when I tried to flash that said "platforms/chibios/platform.mk:270: lib/chibios/os/hal/lib/streams/streams.mk: No such file or directory".

Doing a search lead me to something about submodules. I ran `qmk doctor` at the suggestion of a search result, and it offered to clone the submodules. I said yes and it seems to have installed the submodules, including the one that threw an error, chibios.

After that, I reran the flash command and it seems to have worked. I didn't even have to unplug and replug the mouse, it was working after that.

I was then able to run the Vial appimage and it recognized the adept and showed me the current layout.

## To Do

- Edit in Vial
  - Dragscroll is a macro that clicks once, then does drag scroll on double-tap of the key, then dragscroll is released when clicked again?
  - Other ideas from here: https://www.reddit.com/r/ploopy/comments/1bha9j7/ploopy_adept_with_vial_firmware_quick_write_up_on/