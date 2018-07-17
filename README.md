Display Manager
===============

An open-source Python library which can modify your Mac's display settings.

Includes the library itself, and a command-line API + GUI to use it in pre-specified ways.

You can download the installer [here](./versions/Display Manager v1.0.0.dmg).

## Contents

* [Contact](#contact) - how to reach us
* [System Requirements](#system-requirements) - what you need
* [Purpose](#purpose) - what can be done with Display manager?
* [Limitations](#limitations) - what *can't* be done with Display manager?
* [Get Started](#get-started) - how to get started with Display Manager
* [Overview](#overview) - what is included in this library?
 * [Library](#library)
 * [Command-Line API](#command-line-api)
 * [GUI](#gui)
* [Command-Line Usage](#command-line-usage) - how to use the command-line API
 * [Set](#set)
 * [Show](#show)
 * [Brightness](#brightness)
 * [Rotate](#rotate)
 * [Mirror](#mirror)
 * [Underscan](#underscan)
* [Usage Examples](#usage-examples) - potential use cases for Display Manager
 * [Library Examples](#library-examples)
 * [Command-Line Examples](#command-line-examples)
 * [System Administration Examples](#system-administration-examples)
* [Update History](#update-history)

## Contact

If you have any comments, questions, or concerns, feel free to [send us an email](mailto:mlib-its-mac-github@lists.utah.edu).

## System Requirements

Display Manager only runs on Mac computers. It depends on the Apple-supplied Python 2.7 binary, which lives at `/usr/bin/python` and comes pre-configured with the PyObjC bindings. These bindings allow Python to access the Objective-C methods that perform display manipulations.

If you have replaced the setDefault `/usr/bin/python` binary (which is not generally advised), you should ensure that it has the PyObjC bindings set up correctly.

## Purpose

Display Manager programmatically manages Mac displays, including [display resolution, pixel color depth, refresh rate](#set), [brightness](#brightness), [rotation](#rotate), [screen mirroring](#mirror), and [HDMI underscan](#underscan). Its primary intended purpose is to allow system administrators and developers to automatically configure any number of Mac displays, by use of the command-line scripts and the Display Manager Python library.

Several intended use-cases for Display Manager are elaborated in [usage examples](#usage-examples).

## Limitations

Currently, Display Manager has a few important limitations that are worth noting:

* Can't configure displays at Mac login screen.
 * Recommended workaround: simply configure displays *during* login/logout, or while logged in. Settings persist after logout, so whichever configurations you set during logout will remain active at the login screen.
* Displays are controlled via `DisplayID`, so any interactions with external displays via the command line must be performed by reference to that display's `DisplayID`. (However, any interactions with the main display don't need to specify its `DisplayID` -- it is implied, unless otherwise noted.)
 * Recommended workaround: try manipulating the external displays (e.g. brightness, rotation, resolution, etc.) by their `DisplayID` to see which physical screens they correspond to.

Note: DisplayIDs are metadata descriptions of display capabilities. For more information, see [here](https://en.wikipedia.org/wiki/DisplayID).

## Get Started

First, check that your system satisfies the requirements in [System Requirements](#system-requirements). If you haven't played around with `/usr/bin/python`, it should.

Next, download the latest installer [here](./versions/Display Manager v1.0.0.dmg), or view the archive of all version installers [here](./versions). Included within are two files: `Display Manager.pkg`, and `Uninstall Display Manager.pkg`. To install, click the former and follow the prompts on-screen; to uninstall, do the same for the latter.

For the curious: `Display Manager.pkg` puts the [Display Manager library ("display_manager_lib.py")](#library) in `/Library/Python/2.7/site-packages/`, the [command-line interface ("display_manager.py")](#command-line-api) in `/usr/local/bin`, and the [GUI](#gui) in `/Applications`; `Uninstall Display Manager.pkg` removes all three of these.

Next, see [Overview](#overview) for an idea of what you can do with Display Manager.

## Overview

The Display Manager suite comes in 3 parts: the Display Manager library (display_manager_lib.py), the command-line API (display_manager.py), and the GUI (gui.py).

### Library

The Display Manager library is housed in display_manager_lib.py, which contains the following:

* The `Display` class is a virtual representation of a connected physical display. It allows one to check the status of various display parameters (e.g. brightness, resolution, rotation, etc.) and to configure such parameters.
* The `DisplayMode` class is a simple representation of Quartz's Display Modes. DisplayModes can be sorted, converted to strings, and passed as parameters to various methods which configure the display.
* The `Command` class is called whenever a request is is made of the DisplayManager library. It contains many parameters for display manipulation, and can be manually run as one sees fit.
* The `CommandList` class is a smart container (for `command`s) that allows one to execute several commands at once without command interference

* `getMainDisplay` returns the primary `Display`;
* `getAllDisplays` returns a `Display` for each connected display
* `getIOKit` allows one to manually access the IOKit functions and constants used in Display Manager (usage not recommended -- it's much simpler to go through `Command`s and `Display`s instead, if possible)

### Command-Line API

The command-line API, accessed via display_manager.py, allows you to manually set [display resolution, pixel color depth, refresh rate](#set), [brightness](#brightness), [rotation](#rotate), [screen mirroring](#mirror), and [HDMI underscan](#underscan). See [command-line usage](#command-line-usage) below for more information.

### GUI

First, set the display you'd like to configure the settings for in the displays dropdown menu. Any time you select a display, all of the other menus automatically switch to that display's current settings, but you can refresh them manually by clicking the "refresh" button.

![display dropdown](./resources/gui_1.png)

Next, select the display settings you'd like from the other menus. Note that the brightness, rotation, and underscan sliders default to 0 and cannot be changed if your display does not allow us to access them.

![brightness, rotation, and underscan sliders](./resources/gui_2.png)

To configure screen mirroring, select the display you'd like to mirror, and choose whether to enable or disable it.

![mirroring menu](./resources/gui_3.png)

Finally, select either "Set Display" or "Build Script". If you click "Set Display", the display will be configured to the settings you've selected. If you pick "Build Script", you'll be given a file dialog to save a script which sets your display to these settings automatically whenever run (using the commands seen in `display_manager.py` [interface](#command-line-api)).

![set and build buttons](./resources/gui_4.png)

## Command-Line Usage

The Display Manager command-line API supports the following commands:

```
$ display_manager.py { help | set | show | brightness | rotate | mirror | underscan }
```

The `help` option prints out relevant usage information (similar to this document). You can give any commands as an argument to `help` (e.g. `display_manager.py help mirror`), and you can give `help` as an argument to any commands. Each command has its own help information, which is detailed below:

### Set

The `set` command is used to change the current configuration on a display or across all displays. It does not ask for confirmation; be careful about what you put in here. Running desired settings through `show` beforehand is recommended.

Notes:
* "Pixel color depth" is determined by the number of bits which encode RGB values for a single pixel. For more information, see [here](https://en.wikipedia.org/wiki/Color_depth).
* "HiDPI" , also known as "Retina Display" among Apple products, refers to a high ratio of pixels (or "dots" in "dots per inch"/"DPI") to the physical area they occupy in a display. Fore more information, see [here](https://en.wikipedia.org/wiki/Retina_Display)

| Secondary Commands | Purpose                                                                                      |
|------------|----------------------------------------------------------------------------------------------|
| `help`     | Prints the help instructions.                                                                |
| `closest`  | Set the display to the supported configuration that is closest to the user-supplied values.  |
| `highest`  | Set the display to the highest supported configuration settings.                             |
| `exact`    | Set the display to the specified values **if** they form a supported configuration.          |

| Options                            | Purpose                                                               |
|-----------------------------------|-----------------------------------------------------------------------|
| `-w width`, `--width width`       | Resolution width.                                                     |
| `-h height`, `--height height`    | Resolution height.                                                    |
| `-p depth`, `--pixel-depth depth` | Pixel color depth (default: 32).                                      |
| `-r refresh`, `--refresh refresh` | Refresh rate (in Hz) (default: 0).                                    |
| `'d display', '--display display` | Only change settings for the display with identifier `display`.       |
| `--no-hidpi`                      | Don't use any HiDPI configuration settings.                           |
| `--only-hidpi`                    | Only use HiDPI-scaled configuration settings.                         |

#### Examples

* Set the main display to its highest supported configuration:
```
$ display_manager.py set highest
```

* Set the main display to the closest value to what you want:
```
$ display_manager.py set -w 1024 -h 768
```

* Set the main display to an exact specification:
```
$ display_manager.py set exact -w 1024 -h 768 -p 32 -r 70
```

* Set display `478176570` to use the highest HiDPI-scaled configuration:
```
$ display_manager.py set highest -d 478176570 --only-hidpi
```

### Show

Use the `show` command to learn more about the supported display configurations for your hardware.

| Secondary Commands    | Purpose                                                                                   |
|---------------|-------------------------------------------------------------------------------------------|
| `help`        | Prints the help instructions.                                                             |
| `all`         | Shows all available supported display configuration.                                      |
| `closest`     | Shows the closest supported configuration to the given values.                            |
| `highest`     | Shows the highest available supported display configuration.                              |
| `current`     | Shows the current display configuration.                                                  |
| `displays`    | Shows a list of all attached, configurable displays.                                      |

| Options                            | Purpose                                                               |
|-----------------------------------|-----------------------------------------------------------------------|
| `-w width`, `--width width`       | Resolution width.                                                     |
| `-h height`, `--height height`    | Resolution height.                                                    |
| `-p depth`, `--pixel-depth depth` | Pixel color depth (default: 32).                                      |
| `-r refresh`, `--refresh refresh` | Refresh rate (in Hz) (default: 0).                                    |
| `'d display', '--display display` | Only change settings for the display with identifier `display`.       |
| `--no-hidpi`                      | Don't use any HiDPI configuration settings.                           |
| `--only-hidpi`                    | Only use HiDPI-scaled configuration settings.                         |

#### Examples

* Show the main display's highest supported configuration:
```
$ display_manager.py show highest
resolution: 1600x1200; pixel depth: 32; refresh rate: 60.0; ratio: 1.33:1
```

* Show all connected displays and their identifiers:
```
$ display_manager.py show displays
Display: 478176570 (Main Display)
Display: 478176723
Display: 478173192
Display: 478160349
```

* Show the current configuration of `478176570`:
```
$ display_manager.py show current -d 478176570
```

### Brightness

You can set the brightness on your display with the `brightness` command (assuming your display supports it).

| Secondary Commands    | Purpose                                               |
|---------------|-------------------------------------------------------|
| `help`        | Prints the help instructions.                         |
| `show`        | Show the current brightness setting(s).               |

| Options                | Purpose                                       |
|-----------------------|-----------------------------------------------|
| `-d [display]`, `--display [display]`  | Change the brightness on display `display`.   |

#### Examples

* Show the brightness settings of all displays:
```
$ display_manager.py brightness show
```

* Set the brightness of the main display to its maximum brightness:
```
$ display_manager.py brightness set .4
```

* Set the brightness of display `478176723` to 40% of maximum brightness:
```
$ display_manager.py brightness set .4 -d 478176723
```

### Rotate

You can view and change your display's orientation with the `rotate` command.

| Secondary Commands    | Purpose                                               |
|---------------|-------------------------------------------------------|
| `help`        | Prints the help instructions.                         |
| `show`        | Show the current rotation setting(s).                 |
| `set [value]` | Set display orientation to [value] \(in degrees\).    |

| Options                | Purpose                                       |
|-----------------------|-----------------------------------------------|
| `--display [display]`   | Change the orientation of `display`.        |

#### Examples

* Show the current orientation of all displays (in degrees):
```
$ display_manager.py rotate show
```

* Rotate the main display by 90 degrees (counter-clockwise):
```
$ display_manager.py rotate set 90
```

* Flip display `478176723` upside-down:
```
$ display_manager.py rotate set 180 -d 478176723
```

* Restore display `478176723` to default orientation:
```
$ display_manager.py rotate set 0 -d 478176723
```

### Underscan

The `underscan` command can configure HDMI underscan settings.

Note: HDMI underscan settings can fix displays that under-render images, causing the outer edge of the screen to be left empty. Displays default to underscan 0, and can be set anywhere between 0 and 1, with 1 stretching the display image to its maximum. For more details, see [here](https://support.apple.com/en-us/ht202763).

| Secondary Commands    | Purpose                                               |
|---------------|-------------------------------------------------------|
| `help`        | Prints the help instructions.                         |
| `show`        | Show the current underscan setting(s).                |
| `set [value]` | Set display underscan to [value] between 0 and 1.     |

| Options                | Purpose                                       |
|-----------------------|-----------------------------------------------|
| `--display [display]`   | Change the orientation of `display`.        |

#### Examples

* Set main display to 0% underscan:
```
$ display_manager.py underscan set 0
```

* Set display `478176723` to 42% underscan:
```
$ display_manager.py underscan set .42 -d 478176723
```

### Mirror

The `mirror` command is used to configure display mirroring.

| Secondary Commands | Purpose                                                                  |
|------------|--------------------------------------------------------------------------|
| `help`     | Prints the help instructions.                                            |
| `set`      | Activate mirroring.                                                    |
| `disable`  | Deactivate mirroring.                                                    |

| Options                        | Purpose                                               |
|-------------------------------|-------------------------------------------------------|
| `-d display`, `--display display`  | Change mirroring settings *for* display `display`.        |
| `-m display`, `--mirror display`   | Set the above display to become a mirror *of* `display`.  |

#### Examples

* Set display `478176723` to become a mirror of `478176570`:
```
$ display_manager.py mirror set -d 478176723 -m 478176570
```

* Stop mirroring:
```
$ display_manager.py mirror disable
```

## Usage Examples

Display Manager allows you to manipulate displays in a variety of ways. You can write your own scripts with the [Display Manager library](#library), manually configure displays through the [command-line API](#command-line-api), or access the functionality of the command-line API through the [GUI](#gui). A few potential use cases are outlined below:

### Library Examples

First, import the Display Manager library, like so:

```
from DisplayManager import *
```

Next, say you'd like to automatically set all the displays connected to your computer to their highest resolution. A simple script might look like this:

```
for display in getAllDisplays():
    display.setMode(display.highestMode())
```

Perhaps you'd like your main display to rotate to 90 degrees. The following would work:

```
display = getMainDisplay()
display.setRotate(90)
```

You can use any of the properties and methods of `Display` objects to configure their settings, which is exactly how the [command-line API](#command-line-api) works. You can also configure displays through `Command`s, like this:

```
command = Command("brightness", "set", brightness=.4)
command.run()
```

This would perform the same `Command` as entering the following into the command line:

```
$ display_manager.py brightness set .4
```

For more complex usage, initialize a `CommandList`, which runs several commands simultaneously in a non-interfering pattern. To do so, pass it `Commands` through the `.addCommand(command)` method. An example:

```
commandA = Command("underscan", "set", underscan=.4)
commandB = Command("rotate", "set", angle=180)
commandC = Command("brightness", "set", brightness=1)

commands = CommandList()
commands.append(commandA)
commands.append(commandB)
commands.append(commandC)
commands.run()
```

### Command-Line Examples

In some cases, it may be desirable to configure displays from the command line, whether manually or via a script. Say you'd like a script to automatically set a display to its highest available resolution. The following would do just that:

```
$ display_manager.py set highest
```

But, in many cases, you might want to call several such commands at the same time. Of course, you may write them out line-by-line, but this takes a little longer, and more importantly, running several commands in this way may lead to undesired interference between commands. As such, it is recommended that multiple commands be run like so:

```
$ display_manager.py "set -w 1920 -h 1080" "rotate set 90" "brightness set .5" ...
```

In this way, you may pass in as many commands as you like, and Display Manager will find a way to run them simultaneously without encountering configuration errors.

### System Administration Examples

#### Jamf Pro

"Jamf Pro, developed by Jamf, is a comprehensive management system for Apple macOS computers and iOS devices... \[including\] deploying and maintaining software, responding to security threats, distributing settings, and analyzing inventory data." <sup>[source](https://its.unl.edu/desktop/jamf-casper-suite-faqs/)</sup>

Suppose you'd like all computers in a particular Jamf Pro [scope](http://docs.jamf.com/9.9/casper-suite/administrator-guide/Scope.html) to default to their highest ["retina-friendly" resolution](#set) at maximum brightness at login. You could create a policy

![example Jamf policy](./resources/jamf_1.png)

and add a script containing the following to it

```
#!/bin/bash

# Check to make sure the library and command-line API are both installed
if [[ -e /Library/Python/2.7/site-packages/display_manager_lib.py && \
	-e /usr/local/bin/display_manager.py ]] ; then
	
	display_manager.py "set highest --only-hidpi" "brightness set 1"
```

like so:

![example Jamf script](./resources/jamf_2.png)

For more details about command-line usage, see [here](#command-line-usage); for examples, see [command-line examples](#command-line-examples).

#### Outset

"Outset is a script which automatically processes packages, profiles, and scripts during the boot sequence, user login, or on demand." <sup>[source](https://github.com/chilcote/outset)</sup>

Perhaps you're managing several wall-mounted HDMI displays that are flipped upside-down via Outset, and you'd like them to automatically display right-side-up and set underscan to 50%. You could save the following script to `/usr/local/outset/boot-every/flip.sh`:

```
#!/bin/bash

# Check to make sure the library and command-line API are both installed
if [[ -e /Library/Python/2.7/site-packages/display_manager_lib.py && \
	-e /usr/local/bin/display_manager.py ]] ; then
	
	display_manager.py "rotate set 180" "underscan set .5"
```

For more details about command-line usage, see [here](#command-line-usage); for examples, see [command-line examples](#command-line-examples).

## Update History

| Date       | Version | Update |
|------------|---------|--------|
| 2018-07-13 | 1.0.0 | First edition of full Display Manager. Created the DisplayManager library and the new command-line API, added the ability to run multiple commands at once, added a GUI, and added rotation and HDMI underscan features. |
| 2015-10-28 | 0.1.0 | Legacy iteration of Display Manager. Created command-line API. |
