# My Ploopy Adept Customizations: Ergonomics and How to Solve Drag Scroll Issues with Custom Firmware and Vial

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
3. You don't need Vial to do any of the things I've done, but you do need it for some specific enhancements.

### What is Vial?

"Vial is a feature-rich open-source cross-platform (Windows, Linux and Mac) GUI and a QMK fork for configuring your keyboard in real time." It has more features than VIA, which the Adept is compatible with out of the box. You don't need Vial to do any of things I've done with my adept - you can edit the code, compile it, and flash it using QMK alone, and then configure macros and layers with VIA.

One of the features Vial has that VIA does not is called "tap dance". It lets you configure a key to do one thing when tapped, another when held, etc. I thought I needed tap dance, but I didn't end up using it yet.

## Resources

[Ploopy offers some good advice on compiling and flashing QMK firmware to your device](https://ploopyco.github.io/adept-trackball/appendices/programming/). Start there, but be aware that their instructions are not specific to Vial. You will need their instructions to know how to put the device into bootloader mode so you can flash new firmare onto it.

[I used Vial's instructions to set up my development environment (as of this writing, steps 1-4 at the bottom of this page)](https://get.vial.today/manual/first-use.html#4-my-keyboard-isnt-discovered-or-it-doesnt-work). That includes installing QMK, cloning Vial's fork of QMK, and compiling and flashing their firmware onto your device.

I'll list the file path for every file I changed. All file paths are in the directory of the Vial fork of QMK that you'll clone if you want to follow along.

## Rotated Adept

I work at a computer 40+ hours a week, and I've been feeling some pain in my arms recently. I surmissed this was caused by repetitive stress and resting my arms and wrists on my desk and chair. Your mileage will vary, but negative keyboard tilt  has helped me - tilting the spacebar end of the keyboard up so my wrists tilt down while I'm typing. I figured why not try the same with the Adept. The Adept has a positive tilt, so all I needed to do was rotate it 180 degrees.

An added benefit of doing this is that I have easy access to 4 of the Adept's 6 buttons with my thumb and pinky. I've found I like to move the ball with my fingertips, so it's nice to only have to reach over it for two of the buttons.

[//]: # (TODO)
INSERT PHOTO OF HAND ON TRACKBALL.

### Invert the Trackball

In order to usse the adept like this, we need to invert the trackball's X and optionally Y dimensions. By default, the trackball is inverted in the Y direction but not the X direction. This makes the cursor go up when you rotate the ball away from yourself, and to the right when you rotate it right.

I like this configuration, so I have to invert both the X and Y axes from the setting the Adept ships with.

In `keyboards/ploopyco/madromys/config.h` I commented out the Y invert, but uncommented the X invert and the Dragscroll invert. This makes the ball behave as expected when the device is rotated 180 degrees. [Click here to see that file in context with this change](https://github.com/lortimer/qmk-firmware/commit/eb44d8d10176426cce93231c50c853d556fb4faa#diff-19ad4cb92fd4e9b075244e89ea1cf3f71178a69db2bb84d105ac5930a235bf60R24).

```c
//#define POINTING_DEVICE_INVERT_Y // Don't invert Y so I can use Adept upside down
#define POINTING_DEVICE_INVERT_X // Invert X so I can use Adept updside down
#define PLOOPY_DRAGSCROLL_INVERT // Invert drag scroll direction so I can use Adept updside down
```

Compiling the firmware with these changes in `keyboards/ploopyco/madromys/config.h` file makes the trackball behave as expected when the device is rotated 180 degrees.

### Vial Keymap Layout

Now we need to adjust the keymap in Vial to make sure the keys are laid out how we want. The keymap in vial shows the keys laid out as they are with the device facing the intended direction, and I thought I would make mistakes trying to remap the keys that way. Instead, I wanted to rotate the keys in the keymap that appears in Vial.

I did this by editing the layout in `keyboards/ploopyco/madromys/keymaps/vial/vial.json`. It took a while to figure out how to do this, because the layout system it uses is not documented _anywhere_ that I could find, and it's unlike any other layout system I've ever used. Once I understood it, I realized this system makes it easy to lay out many keys on a row-based keyboardd. I've documented what I learned about it below.

[Click here to see the 180-degree-rotated layout in Github](https://github.com/lortimer/qmk-firmware/commit/eb44d8d10176426cce93231c50c853d556fb4faa#diff-ef1af7aca5dc0ad35b5ec39187cdac9f04f2cdcbc039ba14b09ca5124954d042R12). You may have to scroll down to the vial.json file.

Compiling the firmware with this file makes the adept's buttons appear in the 180-degree-rotated orientation in Vial.

[//]: # (TODO)
INSERT SCREENSHOT OF VIAL

#### How does the layout system work?

In `keyboards/ploopyco/madromys/keymaps/vial/vial.json` `layouts.keymap` is an array of object/string pairs. First comes an object representing coordinatess and size of a button, then string representing which button this object refers to. The string is formatted as "<row>,"column", and I _believe_ this must correspond to the row and column `keyboards/ploopyco/madromys/info.json`, although I haven't seen how that file affects Vial.

As far as I can tell, each keymap object can contain 4 fields, and all of them are optional.
- an `x` coordinate
- a `y` coordinate
- a height value, `h`
- a width value, `w`

##### Height and Width
Each key has a height and width of 1 arbitrary unit, so they appear as squares in Vial by default. 

I let the smaller buttons on the adept be height 1, and I gave the larger buttons a height of 1.5. That way they appear to be about the right size compared to the small buttons.

##### X and Y
`x`'s and `y`'s default values are _relative to the location of the previous button in the array._ `x`'s is also relative to the previous button's width.

A button's `x` coordinate is where the left edge will be, and its `y` coordinate is where the top edge of the button will be. A larger `x` value is farther to the right than a smaller one, and a larger `y` value is farther down than a smaller one. 

The first button in my array is the bottom-left button if you're looking at the Adept as it was intended to be used, but for me is the top-right. I'll draw the buttons moving clockwise from that button. It has (x,y) coordinates of (0,0), which is the default for the first button, but I've set it explicitly.

I want the second button to appear below the first and aligned with it horizontally. Here's where it gets tricky.

The default `x` coordinate for the next button in the array is at the right edge of the previous button by default. That makes sense if you're usually laying out rows of many keys that all touch each other as on a standard keyboard.

You can derive the next button's _default_ `x` coordinate by adding the previous button's width to its `x`. For the second button in my array, the default `x` is  0 (previous x) + 1 (height) = 1. However, I want my second button's left edge to align with the first button's left edge, so I have to subtract the first button's width from it's `x`. 0 (previous button's x) - 1 (previous button's width) gives us an `x` of -1. Remember, this is _relative to the default_ for this button, not absolute, so it will align the second button horizontally with the first.

The default `y` coordinate for the next button is _the same_ as the y coordinate for the previous button. That's because keyboards are typically laid out in rows. I want my second button to be below the previous button, with a little space in between. To make that happen I have to add the height and y coordinate of the previous button and then add an offset. 0 (previous y) + 1.5 (height) + 0.1 (offset) gives the second button a `y` coordinate of 1.6.

 Continuing from there, the next button, one of the small ones, has an `x` of -2.1 so it is a full key plus an offset to the left of the previous button. It has a `y` of 0.52 so its bottom edge aligns with the previous button. We continue in this way until all the buttons are laid out.

## Separate Momentary and Toggle Drag Scroll

The firmware Ploopy provides for the Adept includes a special key definition and custom C code for drag scroll. Activating drag scroll maakes the trackball scroll instead of moving the mouse cursor.

In `keyboards/ploopyco/madromys/config.h`, you can define `PLOOPY_DRAGSCROLL_MOMENTARY` to activate drag scroll only while the key is held down, called "momentary" drag scroll. Or you can comment that line out to make the drag scroll button toggle the feature on and off.

I wanted both. I usually prefer to use momentary drag scroll because it gives me the flexibility to quickly scroll a little bit while also clicking on things. If I'm scrolling a very long document and not using the mouse otherwise, I may want to toggle it on to give my hand a break from holding the button.

We need changes to a few files to do this. [All of the changes required can be found in this commit on Github.](https://github.com/lortimer/qmk-firmware/commit/413f8951fa62bf94c98e9707686ef2130bab34f0)

First, we need a new keycode to represent momentary drag scroll. We'll treat the existing  keycode as the toggle drag scroll. Custom keycodes are enumerated in `keyboards/ploopyco/ploopyco.h` in an enum called ploopy keycodes. We'll add `DRAG_SCROLL_MOMENTARY` at the end as in the code snippet below. 

```c
enum ploopy_keycodes {
    DPI_CONFIG = QK_KB_0,
    DRAG_SCROLL,
    DRAG_SCROLL_MOMENTARY,
};
```

Now we can use our new keycode in the code that toggles drag scroll in `keyboards/ploopyco/ploopyco.c`. Before we change anything, observe in the code snippet below that the `process_record_kb` function checks whether `PLOOPY_DRAGSCROLL_MOMENTARY` is defined in `keyboards/ploopyco/madromys/config.h`. If it is, it sets a variable called `is_drag_scroll` to activate dragscroll if the key is currently held down. If not, it toggles that variable. 

```c
bool process_record_kb(uint16_t keycode, keyrecord_t* record) {
    // ...other code here
    if (keycode == DRAG_SCROLL) {
    #ifdef PLOOPY_DRAGSCROLL_MOMENTARY
            is_drag_scroll = record->event.pressed;
    #else
            if (record->event.pressed) {
                toggle_drag_scroll();
            }
    #endif
    }
    // ...other code here
}
```

We can change this section of the function to look for our new keycode instead of looking for `PLOOPY_DRAGSCROLL_MOMENTARY`, as in the code snippet below

```c
if (keycode == DRAG_SCROLL) {
    if (record->event.pressed) {
        toggle_drag_scroll();
    }
}

if (keycode == DRAG_SCROLL_MOMENTARY) {
    is_drag_scroll = record->event.pressed;
}
```

Finally, we need to tell Vial about our new keycode so it will be available to assign to a button when we're editing the keymap. We add it to the `custom keycodes` array in `keyboards/ploopyco/madromys/keymaps/vial/vial.json`, and give it a name, title, and short name as in the following code snippet.

```c
  "customKeycodes": [
    {"name": "DPI Config", "title": "DPI Config", "shortName": "DPI"},
    {"name": "Toggle Drag Scroll", "title": "Toggle Drag Scroll", "shortName": "Drag\nScrl"},
    {"name": "Momentary Drag Scroll", "title": "Momentary Drag Scroll", "shortName": "MDrag\nScrl"},
  ],
```

## Making drag scroll work consistently on MacOS

[//]: # (TODO: Mailto link)
I use MacOS for work. You can send your condolences to info@ryanheisler.com. When I switched to using the Adept on Mac I noticed that drag scroll would work sometimes and not others. I would have to press and hold momentary drag scroll two to five times before it would scroll.

### Investigation

When it didn't work, the mouse cursor would not move. The drag scroll code prevents the cursor from moving, so I took this as evidence that the Adept knew I was trying to scroll but the operating system wasn't recognizing the scroll events coming from the Adept.

[//]: # (TODO: Answer this question with my blog post: https://www.reddit.com/r/ploopy/comments/1jkm4on/adept_scrolling_works_intermittently/)

Searching the web, [I found someone complaining of a similar issue in the Ploopy subreddit two years ago](https://www.reddit.com/r/ploopy/comments/1cp57m6/scrolling_issue_scroll_wheel_and_drag_scroll_with/) Desspite posting there and in the QMK subreddit, they never got a resolution.

### Failed Attempt to Use Mouse Wheel Events

`keyboards/ploopyco/ploopyco.c` has a function called `pointing_device_task_kb`, which processes mouse updates, and it's where the drag scroll functionality is actually implemented. It makes drag scroll work by following the steps below. [You can see how this is done here on Github.](https://github.com/lortimer/qmk-firmware/blob/413f8951fa62bf94c98e9707686ef2130bab34f0/keyboards/ploopyco/ploopyco.c#L141)

1. Get `x` and `y` from the current mouse report. These represent vertical and horizontal changes in the mouse cursor's position since the last time the position was updated.
2. Divide these values by a configured divisor (to allow the user to control how fast or slow the drag scroll goes)
3. Assign these values to the mouse report's `v` (vertical) and `h` (horizontal) fields, which the operating system will interpret as a scroll event.
4. Set the mouse report's `x` and `y` to 0 to prevent the mouse cursor from moving.

I have a third-party mouse with a scroll wheel and it works just fine on a Mac. I figured this method of scrolling using the mouse report's `v` and `h` might be poorly supported on MacOS, and that leads to it working intermittently. Looking at the standard keycodes available in Vial, I noticed that there are some representing the mouse wheel going up, down, right, or left. These are no doubt included so you can make a key on your keyboard scroll when pressed. I thought they might help me scroll in a different way that's better supported by MacOS.

To make this happen, I implemented two new custom keycodes - DRAG_SCROLL_MACOS and DRAG_SCROLL_MOMENTARY_MACOS. Instead of setting the `v` and `h` on the mouse report, they called a function called a function to tap the correct keycode based on how the trackball moved. [You can see how I implemented this here, in `keyboards/ploopyco/ploopyco.c` starting on line 172](https://github.com/lortimer/qmk-firmware/commit/36f290f1b2dc5a9e00a83faba2dead534d5a6821#diff-5f763322c53918e554122f33ae3ca19598084b9e5356cfdef6efa9625bab6c7cR170)

This didn't work. It had the same problem where scrolling would only work some of the time on MacOS, but it was worse than before because when scrolling did work it was choppier and harder to control.

### Scrolling in Multiple Directions

When I tried using my new scrolling behavior on a Mac, I noticed something new. When I was scrolling vertically it would not scroll horizontally no matter how much I rotated the trackball sideways. Similarly, if I was scrolling horizontally it wouldn't scroll vertically. The reddit post I linked above mentioned that scrolling would work again if they waited a few seconds with the button held down, so I tried this. Lo and behold I could scroll in my desired direction again if I waited a few seconds, even if I was switching from horizontal to vertical.

I investigated further - could I scroll in multiple dimensions at once using the trackpad built into the Macbook? _No_! And that wass true across appss. Apparently, MacOS prevents you from scrolling vertically and horizontally at the same time.