---
title: Manual bed leveling on Ender3
layout: single
author_profile: true
read_time: true
comments: null
share: true
categories:
  - 3DPrinting
tags:
  - bedLeveling
  - 3dprinting
  - ender3
  - scripting
  - cura
  - openscad
---

Either if you are new with 3D printing, or you are a veteran, this post can show you something new!

Let's start with the basics!

I assume you are familiar with the [basics](https://www.youtube.com/watch?v=5eqTmb01cBk).
This method is pretty good, you do the paper method, and start to print an stl file, and tune your printer while it prints.

BUT! If your last rectangle is crappy you need to reprint the whole thing. And it is slow.
And while you check your output the nozzle is cooling down. 
And what if I just want to do a quick sanity check because I maybe touched the front right knob?
Also there are not a lot of options in the premade stl-s.   

I thought at first, that I will slice every rectangle separately from a recommended stl.
But I'm (as most of the programmers) lazy, and hate manual works. So started some thinkering.

If I have an openScad file and I can generate stls from it. 
And if I can get out the parameters from my cura and can call CuraEngine from a script...

This example will only work out of the box from macOS, but you can get the idea easily. 
I think it is almost no-brainer to port to linux.
And probably with the linux subsystem (or some console emulators like [cmder](https://cmder.net/)) it can be executed with little modifications on windows too. 

I have [OpenSCAD (v.2015-03-3)](https://www.openscad.org/) and [Cura v4.0](https://ultimaker.com/en/products/ultimaker-cura-software) installed.
First things first you will need to find your applications executables.
 - For Cura: `/Applications/Ultimaker\ Cura.app/Contents/MacOS/CuraEngine`
 - For OpenSCAD: `/Applications/OpenSCAD.app/Contents/MacOS/OpenSCAD`
 - (ofc these can differ but with tab completition they are relatively easy to find)


### Get OpenScad to generate an stl
I get a bed-leveling (scad) file from [thingiverse (thx pgreenland)](https://www.thingiverse.com/thing:34558).
The original file assumed a bottom left (0,0) coordinate, and a smaller printbed.
It also did some perimeter (I think this is the good word and not the parameter as in the original file says but who cares?).

I just tuned the variables, added a translation to make the middle of the bed to (0,0), and added a bunch of ifs to it.

Before that I learned that I can call openScad from the command line and add variables to it dynamically. 
And I tried out that if a variable is not set it will be false (but generates a warrning). See the file [here](https://github.com/tg44/3d-tools-and-things/blob/master/bed-leveling-generator-script/bed_leveling.scad)!

One command looks like:
`/Applications/OpenSCAD.app/Contents/MacOS/OpenSCAD -o "all.stl" -D col=true -D row=true bed_leveling.scad`

### Get CuraEngine to generate a gcode
This was a lot messier thing. 
If you think that Cura has some kind of json/ini that can be find and feeded to the CuraEngine to generate a gcode from the given stl;
you are wrong.
I (as a programmer) don't really understand this design decision, or why this has no structure, or why it didn't fall back to already defined values. 
(I think they are developing one of the best slicer, and don't understand why they didn't have a "nicer api" requirement...)

How to get the needed parameters from Cura to later feed them to CuraEngine?
 - Start Cura (the GUI)
 - Choose your favorite Cura profile
 - Slice ANY stl with it (without moving/scaling/modifying)
 - Open the logs (`tail -n 500 ~/Library/Application\ Support/cura/4.0/cura.log`)
 - Copy out the relevant lines (yes you start to feel it)
 - Double-check that all 3 blocks are copied out (the machine start/end not needed, that can be added back at the end of this process and can be copied from the ui too)
 - Mass-format the output (`sed -i '' 's/-s/\\\'$'\n-s/g' normal_cura_settings.txt`)
 - Fixup the last lines
   - you will have something like this at the end:
     ```
     -s jerk_support_bottom="20" -g -e0 -l "0" \
     -s extruder_nr="0"
     ```
    - you want to change it to something like this:
      ```
      -s jerk_support_bottom="20" \
      -g -e0 -l input.stl \
      -s extruder_nr="0" \
      -o output.gcode
      ```
 - (add the machine start/end if you left it out when copy-pasting)
 - add this line before the whole thing: `/Applications/Ultimaker\ Cura.app/Contents/MacOS/CuraEngine slice \`
 - try to run it on the same stl as you sliced before (if it failes try to find messedup lineendings)
 - compare the outputs (ideally they are mostly the same or at least they really look the same)
   - you can do this with VisualStudioCode or intelliJ or `diff`

### Assembly
We have a scad file, we need to call it in a double for loop; we will have every col/row and every rectangle separately too.
Get all of these stl-s and call CuraEngine on it.
Remove the cooling codes from the end of the gcodes (so we can run them separately right after each other).

The full script is [here](https://github.com/tg44/3d-tools-and-things/blob/master/bed-leveling-generator-script/bed.sh) with all my cura settings for the bed leveling. 
(I don't use build-plate adhension for this, and I deleted the initial line on the left side from the code.)

The output is also [there](https://github.com/tg44/3d-tools-and-things/tree/master/bed-leveling-generator-script/bed_leveling) if you trust in my settings/gcode. (!!! You are responsible for any code you upload to your printer. !!! )

(I'm thinking of porting these tools to docker containers so they can run without knowing the host platform. If you are interested with this, lets talk in the github-repo issue section. Also if you have any idea; issues and PRs are welcome!)

### Usage
Move the folder to your sd-card. 
Print it, when your bed is not leveled! 
Don't forget to manually cool-down your printer before turning it off!
