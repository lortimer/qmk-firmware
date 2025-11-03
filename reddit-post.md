#My Ploopy Adept Customizations: Ergonomics and How to Solve Drag Scroll Issues with Custom Firmware and Vial

I recently got a Ploopy Adept for the customization options and ergonomics, and I'm super happy with it. I've made a few customizations I want to share in case it can help others get the most out of their QMK trackballs.

## Contents
1. A note about customizing the firmware
2. Resources I used to figure this out
3. Rotate the Adept 180 degrees for negative tilt and easy access to more buttons
4. Have access to both momentary and toggled drag scroll
5. Make dragscroll work reliably on MacOS

## Customizing your firmware

For all of these customizations I used Vial and Vial-compatible firmware. __The Ploopy Adept does not ship with Vial-compatible firmware__. You will have to compile and flash the firmware onto your device on your own.

1. You do this at your own risk. I'm not responsible if you brick your trackball (but [Ploopy can has a way to flash new firmware if you do brick it](https://ploopyco.github.io/adept-trackball/appendices/programming/#putting-the-ploopy-device-into-bootloader-mode-if-its-bricked))
2. I cannot help you do anything I've done. I'll give as much info as I can in this post, but it's up to you to figure it out.

## Resources

[Ploopy offers some good advice on compiling and flashing QMK firmware to your device](https://ploopyco.github.io/adept-trackball/appendices/programming/). Start there, but be aware that their instructions are not specific to Vial. You will need these instructions to know how to put the device into bootloader mode so you can flash new firmare onto it.

[I used Vial's instructions to set up my development environment (as of this writing, steps 1-4 at the bottom of this page)](https://get.vial.today/manual/first-use.html#4-my-keyboard-isnt-discovered-or-it-doesnt-work). That includes installing QMK, cloning Vial's fork of QMK, and compiling and flashing their firmware onto your device.

I'll list the file path for every file I changed. All file paths are in the directory of the Vial fork of QMK that you'll clone if you want to follow along.

### What is Vial?

 "Vial is a feature-rich open-source cross-platform (Windows, Linux and Mac) GUI and a QMK fork for configuring your keyboard in real time." It's a fancier version of VIA, which the Adept is compatible with out of the box. You don't need Vial to do any of things I've done with my adept - you can edit the code, compile it, and flash it using QMK alone, and then configure macros and layers with VIA.

However, I used Vial for a couple reasons: 1. I didn't know I didn't need it when I started, and 2. It has a few features VIA doesn't. One is called "tap dance", which lets you configure a key to do one thing when tapped, another when held, etc. I thought I needed tap dance but I didn't end up using it yet.

## Rotated Adept

I work at a computer 40+ hours a week, and I've been feeling some pain in my arms recently, I thought because of repetitive stress and resting my arms and wrists on things. Your mileage will vary, but what has worked for me is negative keyboard tilt - tilting the spacebar end of the keyboard up so my wrists tilt down while I'm typing. I figured why not try the same with the Adept, and conveniently all I needed to do was rotate it 180 degrees.

An added benefit of doing this is that I have easy access to 4 of the Adept's 6 buttons with my thumb and pinky. I've found I like to move the ball with my fingertips, so it's nice to only have to reach over it for two of the buttons.

[//]: # (TODO)
INSERT PHOTO OF HAND ON TRACKBALL.

### Invert the Trackball

By default, the trackball is inverted in the Y direction but not the X direction. This makes the cursor go up when you rotate the ball away from yourself, and to the right when you rotate it right.

I'm going to use this thing upside down, so I have to invert the inversion.

In `keyboards/ploopyco/madromys/config.h` I commented out the Y invert, but uncommented the X invert and the Dragscroll invert. This makes the ball behave as expected when the device is rotated 180 degrees. [Click here to see that file in context with this change](https://github.com/lortimer/qmk-firmware/commit/eb44d8d10176426cce93231c50c853d556fb4faa#diff-19ad4cb92fd4e9b075244e89ea1cf3f71178a69db2bb84d105ac5930a235bf60R24).

```c
//#define POINTING_DEVICE_INVERT_Y // Don't invert Y so I can use Adept upside down
#define POINTING_DEVICE_INVERT_X // Invert X so I can use Adept updside down
#define PLOOPY_DRAGSCROLL_INVERT // Invert drag scroll direction so I can use Adept updside down
```

### Vial Keymap Layout

To make this work, you have to remap all the buttons, and ideally rotate how the buttons appear in the Vial app so you don't have to rotate them in your mind while you're assigning buttons.

I did this by editing the layout in keyboards/ploopyco/madromys/keymaps/vial/vial.json. This was super annoying because the layout system is not documented _anywhere_ that I could find, and it's unlike any other layout system I've ever used.

[Here's the layout I use in Github](https://github.com/lortimer/qmk-firmware/commit/eb44d8d10176426cce93231c50c853d556fb4faa#diff-ef1af7aca5dc0ad35b5ec39187cdac9f04f2cdcbc039ba14b09ca5124954d042R12). You may have to scroll down to the vial.json file.

Compiling the firmware with this file as it is makes the adept's buttons appear in the 180-degree-rotated orientation in Vial.

[//]: # (TODO)
INSERT SCREENSHOT OF VIAL

#### How does the layout system work?
If you want to change the layout, here's how it works.

`keymap` is an array that alternates between objects and strings - off to a great start. They come in pairs, first an object and then a corresponding string. In my keymap The object contains three things:
- an `x` coordinate
- a `y` coordinate
- a height value, `h`

The string is formatted as "<row>,"column", and I _believe_ this must correspond to the row and column keyboards/ploopyco/madromys/info.json, although I haven't seen how that file affects Vial.

##### Height
Height is the easiest to explain. Each key has a height and width of 1 arbitrary unit by default, so they appear as squares in Vial by default. 

I let the smaller buttons on the adept be height 1, and I gave the larger buttons a height of 1.5. That way they appear about the right relative size to the small buttons.

##### X and Y
`x` and `y`, on the other hand, are _relative to the location and size of the previous button in the array._

The first button in my array is the bottom-left button if you're looking at the Adept as it was intended to be used, but for me is the top-right. I'll draw the buttons moving clockwise from that button. It has (x,y) coordinates of (0,0). Makes sense.

A button's `x` coordinate is where the left edge will be, and a positive `x` value is relatively to the right. A button's `y` coordinate is where the top edge of the button will be, and a positive `y` value is relatively down.

I want the second button to appear below the first and aligned with it horizontally. Here's where it gets tricky.

The default `x` coordinate for the next button is at the right edge of the previous button by default. To understand it, add the width of the previous button to the x coordinate of the previous button. If you want your button to have a different x coordinate, you have to set `x` to an offset value. I want my second button's left edge to align with the first button's left edge, so I have to subtract the first button's width from it's x coordinate. Therefore, the `x` for the second button is -1.

The default `y` coordinate for the next button is the _same_ as the y coordinate for the previous button. I want my second button to be below the previous button, with a little space in between. To make that happen I have to add the height and y coordinate of the previous button and add an offset. 0 (previous y) + 1.5 (height) + 0.1 (offset) gives the second button a y coordinate of 1.6.

I understand that this system works well for laying out a row-based keyboard of mostly 1x1 buttons, and isn't meant for something like the Adept. You only have to change the defaults for the few buttons that are different sizes. I just wish it was documented somewhere.

## Separate Momentary and Toggle Drag Scroll

The firmware Ploopy provides for the Adept includes a special key code and custom C code for Drag scroll. We need it to change the trackball's behavior from moving the mouse cursor to scrolling.

In `keyboards/ploopyco/madromys/config.h`, you can define `PLOOPY_DRAGSCROLL_MOMENTARY` to make the drag scroll only work while the key is held down. Or you can comment that line out to make the drag scroll key toggle the feature on and off.

I wanted both. I prefer the flexibility to quickly scroll a little bit by holding the button down, but if I'm scrolling a very long document and not using the mouse otherwise, I want to toggle it on to give my hand a break.

