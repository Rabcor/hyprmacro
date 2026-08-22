# hyprmacro
hyprmacro is a shell script that interfaces with `hyprctl` in order to create macros. This is a script i made for myself but I figured I might as well share it. The script is mostly complete with only some minor conveniences missing, feel free to report any issues you encounter.

This script requires hyprland version 0.55+, and it also must be configured with `~/.config/hypr/hyprland.lua` rather than the old `~/.config/hypr/hyprland.conf`.

Here is an [older incomplete version](https://gist.github.com/Rabcor/a98c466f3d8268bbc371f92b268ec6b7) that supports the old hyprland config.
# Installation
Move the script to either `/usr/bin` or `~/.local/bin` and make it executable.

# Usage
The script has a help dialog which outlines it's features

```
Example: hyprmacro -F -a 'class:some_app' -m abort -r 5-20 -l 0 -k mouse1
Arguments are processed sequentially.
Options:
	-a		Send to specific app (Ex: -a address:0x557b19812720)
	-F		Execute only if target app is currently in focus (must be called before -a)
	-m		Set parallel execution mode:
				parallel	 - Run macro in parallel with other running macros.
				override-all - If any macro is already running, terminate them and execute.
				cancel-all	 - If any macro is already running, terminate them and exit.
				queue		 - If this macro is already running, wait for prior execution to finish before starting.
				override	 - If this macro is already running, terminate it and execute.
				cancel		 - If this macro is already running, terminate it and exit. (Default)
				abort		 - If this macro is already running, exit.
	-k		Press key once (Ex: -k s)
	-d		Key down (Ex: -d s)
	-u		Key up (Ex: -u s)OOO
	-m		Move cursor to coordinates (Ex: -m 1000x500)
	-M		Report current cursor coordinates and copy them to clipboard
	-l		Loop macro n times, 0 to loop until macro is invoked again
	-r		Add randomized range of milliseconds delay between all keystrokes (Ex: -r 5-10)
	-s		Add milliseconds delay until the next keystroke 
	-S		Add milliseconds delay between all keystrokes (Ex: -s 100)	
	-T		Tab into the target application automatically when pressing keys (workaround)
	--lang	Execute macro with specific keyboard layout (Example: --lang us)
```

The -a option will try to interpret any string that does not contain a ':' as either a class or title before giving up so using proper formatting is not strictly necessary.

`-a proton-game` or `-a game` are also specially handled cases that will be interpreted as xdg and content type tags respectively, otherwise the normal syntax for hyprctl applies for this option.

Some examples:
```
# Press a
hyprmacro -k a

# Hold a for 10 seconds
hyprmacro -d a -s 10000 -u a

# Right click
hyprmacro -k mouse_right

# Right click
hyprmacro -k mouse2

# Spam left click until re-invoked
hyprmacro -l 0 -k mouse1

# Send a key press to featherpad
hyprmacro -a 'class:featherpad' -k a

# Send a key press to featherpad (lazy, the script will try to interpret the app as either a title or class in this case)
hyprmacro -a featherpad -k a
```

The most effective way to use these to for instance create a macro in a gamae is to use `hyprland.lua` to configure keybinds, here are som ereal life examples.

```
--This fixes an issue where in a lot of games you cannot open the console when running them over proton on a non-english keyboard.
hl.bind("dead_abovering", hl.dsp.exec_cmd(hyprmacro .. "-F -a proton-game -u dead_abovering --lang us -k grave"), { non_consuming = true })

--This allows you to press c to slide and mouse forward (mouse 5) to do a heavy attack. On weapons that have quick heavy attack animations, the queue mode allows them to be spammed at a speed just milliseconds off from the theoretical maximum speed.
hl.bind("c", hl.dsp.exec_cmd(hyprmacro .. '-F -a "class:steam_app_1361210|darktide.exe" -m abort -u mouse2 -u mouse1 -u s -d w -d SHIFT+SHIFT_L -s 200 -d CONTROL+CONTROL_L -u SHIFT+SHIFT_L -s 750 -u CONTROL+CONTROL_L'), { non_consuming = true })
hl.bind("mouse:276", hl.dsp.exec_cmd(hyprmacro .. '-F -a "class:steam_app_1361210|darktide.exe" -m queue -d mouse_left -s 360 -u mouse_left -s 10'), { non_consuming = true })

--Macro to shake vending machines that are side by side in Abiotic Factor
hl.bind("SHIFT + o", hl.dsp.exec_cmd(hyprmacro .. "-a class:steam_app_427410 -l 0 -d e -s 2000 -u e -d a -s 350 -u a -d e -s 2000 -u e -d d -s 350 -u d"), { non_consuming = true })
```

As a general rule, when using `-F`, `-a` and/or `-m` options, always use them as your first arguments for efficiency's sake. Because arguments are handled sequentially, and these particular arguments will apply conditional exit clauses so the sooner you put this argument on the command line, the sooner the script will exit in scenarios where it should not trigger.
