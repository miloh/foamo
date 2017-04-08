fomo foamcutter of machining out  
================================

5 Axis CNC foam cutter with renee wire run by a computer with machinekit
X1,X2, Y1,Y2, A1

project schematic
wiring schematic

(+ Notes on Machinekit and BeagleG)
===================================
* The beaglebone black exposes  92 physical  pins from the Sitara SOC that is its beating heart.
* 20 of these are the usual power ground and whatnot
* 74 pins are usable by various interfaces that are somewhat reconfigurable in software (subset lists of optional outputs exist for each physical pin)

For FOAMO we will need some fast PRU enabled pins:
=================================================
10 pins   -  5 Axis Machine X1, Y1,X2, Y2, A(each stepper requires 2 lines: DIR & STEP)
01 pin     -  switch (wire switch control PWM?)
04 pin      -  endstop switches
16 pins total

We also need to reserve pins for the following functions:
---------------------------------------------------------
* estop(1) 
* temp sensing(1?) 
* control pad (rotary control, buttons, power status led, heat status led)
  - eQEP0 hardware encoder switch (4)
  - eQEP0 hardware encoder siwtch (4)
  - I2C power/heat status led (1)


Some other options:
04  pins       - eQEP0  hardware encoder switch
04  pins      - eQEP1  hardware encoder switch, no hdmi with this
04  pins       - eQEP2  hardware encoder switch, no hdmi with this
01 pin       - I2C (control panel lights)
03 pins       - SPI display feedback 

Henner has PRU configured to drive 8 motors all at 1Mhz in parallel.  Steppers
don't even run above some 20-30kHz..   The generic_pru_driver that is part of
machinekit may have enough performance.  

===========================
ARM.Beaglebone.Xylotex/setup.sh
===========================
sudo $(which config-pin) -f - <<- EOF

        P8.07   out     # gpio2.2       Enable System
        P8.10   in      # gpio2.4       XLIM
        P8.11   out     # gpio1.13      X_Dir
        P8.12   out     # gpio1.12      X_Step
        P8.13   out     # gpio0.23      PWM0/SPINDLE
        P8.14   in      # gpio0.26      YLIM
        P8.15   out     # gpio1.15      Y_Dir
        P8.16   out     # gpio1.14      Y_Step
        P8.18   in      # gpio2.1       ZLIM
        P8.19   out     # gpio0.22      PWM1
        P9.14   out     # gpio1.18      PWM2
        P9.15   out     # gpio1.16      Z_Step
        P9.23   out     # gpio1.17      Z_Dir
#       P9.17   out     # gpio0.5       SCS
#       P9.18   in      # gpio0.4       SDI
#       P9.21   out     # gpio0.3       SDO
#       P9.22   out     # gpio0.2       SCK
        P9.13   out     # gpio0.30      A_Dir
        P9.11   out     # gpio0.31      A_Step
        P8.09   in      # gpio2.5       STOPin
EOF
*************************************************


======================================================
github.com:hzeller/beagleg/hardware/BUMPS/beagleg-pin-mapping.h
=======================================================
// see motor-interface-constants.h for available PINs

// This contains the defines the GPIO mappings for BUMPS
// https://github.com/hzeller/bumps

#define MOTOR_1_STEP_GPIO  PIN_P9_18  // motor 1
#define MOTOR_2_STEP_GPIO  PIN_P9_17  // motor 2
#define MOTOR_3_STEP_GPIO  PIN_P9_21  // motor 3
#define MOTOR_4_STEP_GPIO  PIN_P9_42  // motor 4
#define MOTOR_5_STEP_GPIO  PIN_P9_22  // motor 5
#define MOTOR_6_STEP_GPIO  PIN_P9_26  // (extern 6)
#define MOTOR_7_STEP_GPIO  PIN_P9_24  // (extern 7)
#define MOTOR_8_STEP_GPIO  PIN_P9_41  // (extern 8)

#define MOTOR_1_DIR_GPIO   PIN_P8_16  // motor 1
#define MOTOR_2_DIR_GPIO   PIN_P8_15  // motor 2
#define MOTOR_3_DIR_GPIO   PIN_P8_11  // motor 3
#define MOTOR_4_DIR_GPIO   PIN_P9_15  // motor 4
#define MOTOR_5_DIR_GPIO   PIN_P8_12  // motor 5
#define MOTOR_6_DIR_GPIO   PIN_P9_23  // (extern 6)
#define MOTOR_7_DIR_GPIO   PIN_P9_14  // (extern 7)
#define MOTOR_8_DIR_GPIO   PIN_P9_16  // (extern 8)

#define MOTOR_ENABLE_GPIO  PIN_P9_12  // ENn
#define MOTOR_ENABLE_IS_ACTIVE_HIGH 0  // 1 if EN, 0 if ~EN

#define AUX_1_GPIO         PIN_P9_11  // AUX_1 "Aux, Open Collector"
#define AUX_2_GPIO         PIN_P9_13  // AUX_2 "Aux, Open Collector"

#define PWM_1_GPIO         PIN_P8_9   // PWM_1 "Power PWM"
#define PWM_2_GPIO         PIN_P8_10  // PWM_2 "Power PWM"
#define PWM_3_GPIO         PIN_P8_7   // PWM_3 "PWM, Open Collector"
#define PWM_4_GPIO         PIN_P8_8   // PWM_4 "PWM, Open Collector"

#define IN_1_GPIO          PIN_P8_13  // END_X
#define IN_2_GPIO          PIN_P8_14  // END_Y
#define IN_3_GPIO          PIN_P8_17  // END_Z
*********************************************************************

gEDA Terminology & Description
------------------------------

gEDA is a suite of tools 
* gschem - electronic schematic editor that has some operational similarity to old versions of OrCAD
* gnetlist outputs a number of netlist formats from gschem, part of the sim workflow 
* schdiff - works as a git difftool and uses imagemagick to generate visual diffs of gschem schematics
* refdes\_renum a tool for giving unique 'reference designations' to symbols in a sch file
* gaf stands for gschem & friends, an eponymous cli for use with gschem & friends.
* spice tools -- thofficial spice package for use with geda-gaf. with complete symbols, gnetlist creates spice compatible netwlist
* pcb, aka PCB or what I'll call gEDA's PCB - a powerful and fun floss circuit layout program
* other projects are anything I've forgotten

Project Hierarchy and using gnu Make
------------------------------------
The main directory contains templates for a schematic built using gschem: and
layout files for geda's PCB:

```
*.sch
```

```
*.pcb
```

These all get processed using Make, with the included Makefile. Note this
Makefile hasn't been tested with more than a few versions of gnu make. 

To use the makefile, you run make and supply a goal. The following list is from
a system with tab completion, which supplies the user with the list of goals
available from the makefile.
Some of these will require user actions, like providing the correct filetypes
in the local directory, and
ensuring they use the cvs-based keywords that sed will process the files with,
and using git to tag versions.

````
clean                  gnetlist-bom          hackvana-gerbers.zip  Makefile              
schematic-template.sch osh-park-gerbers.zip  pdf                   gerbers               
hackvana-gerbers       list-gedafiles        layout-template.pcb 
osh-park-gerbers       pcb-bom               ps
````

This makefile uses the commonly available sed and echo, the less available 'gaf'
project from geda, and is intended for use by a hardware designer using gschem
for schematic capture and geda pcb for layout. 

Finally, it also uses git, specifically the git-tag comand to template the
keywords in the schematic and layout templates. The templates should be availabe
for checkout from the early revisions of the project). Versions released for
manufacturing should include annotated version tags using semver (vXX.YY.ZZ,
XX=major YY=minor ZZ=patch)

Bug reports are welcome, create issues on github or send them to miloh at
froggytoad dot net

Git submodules
--------------
This project uses git submodules for libraries of schematic parts and
footprints. 

First, update the git submodules after cloning the project and regularly during
development unless you want to freeze the schematics and parts to a specific
branch (which may totally make sense for some completed projects).

```
git submodule update --init --recursive
```

Updating submodules is important to remember, because when checking out dev
branches or earlier tags of the project, you will have to update the submodules
to get the correct version of parts (symbols and footprints) used during
development. The following command should also be used after checking out
earlier versions to keep the project synced

```
git submodule update --init  --recursive
```

Note that gschem should use a local file 'gafrc' with a line in scheme that
configures the directory for local symbol libraries.

PCB preferences must be changed to find local footprints, I do this in the PCB
gui currently but I imagine there are other ways.

Using schdiff with git's difftool
---------------------------------
schdiff allows the user to compare schematics from different versions.

example showing a diff from the current HEAD to 30 commits back:

```
git difftool -x schdiff HEAD~30 project.sch
```

Squashing the history after copying the template
------------------------------------------------

The history of this template project doesn't need to be part of the history a specific project that uses it
To squash the history, you can rebase back to the first commit.

```
git rebase -i `git rev-list --max-parents=0 HEAD` 
```
Squash everything but the top, change the comments as you see fit. Read up on git rebase 

```
git rebase --help
```
