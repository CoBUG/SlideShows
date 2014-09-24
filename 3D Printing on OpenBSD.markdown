# 3D Printing on OpenBSD

**or** how to port python apps to OpenBSD!

---

## Overview of 3D Printing

  - Modeling (or finding) what you are going to print.
  - Slicing.
  - Printing.

---

## Modeling

This is an entire talk in and of itself, however, you have lots of choices!

|Name|Open|Ease of Use|OPFF*|
|---|---|---|---|
|[Blender](http://www.blender.org/)|Yes|Easy, don't let the UI scare you!|Yes|
|[OpenSCAD](http://www.openscad.org/)|Yes|Code Oriented|Yes|
|[Sketchup](http://www.sketchup.com/)|No|Very Easy|Yes (with plugin)|

** *Output Printer Friendly Files**

---
## Finding things to print

Many objects have already been created, so search first, create later!

  - [Thingiverse](http://www.thingiverse.com/)
  - [Repables](http://repables.com/)
  - [Magicube](http://www.magicu.be/)

---

## Slicing

What the heck is slicing? Why do we need it?

Have you ever done the "Robot Programming" exercise?

---

## Slicing (cont...)

Slicing is basically cutting a 3D model into commands that the printer turns into physical actions (think Robot Programming.. go forward 3 steps,
 turn left).

Generally slicers produce [G-Code](https://en.wikipedia.org/wiki/G-code) which is the final format printing apps like OctoPrint use.
---

## Reasons for slicing

  - Controlers on 3D printers are fairly basic, can't run large codebase.
  - 3D objects need to be represented in a linear set if instructions.
  - ![Roberto](http://img1.wikia.nocookie.net/__cb20110619182621/en.futurama/images/9/96/Futurama_roberto.png)

Roberto approves!
---

## Robot Programming

**Robot Code**
```
forward(3);
turn('Left');
```

**G-Code**
```
G01 X114.395 Y115.288 F1080.000
```

Not really a one to one - but you get the idea :D


---
## Slicing Applications

|Name|Open|Ease of Use|
|---|---|---|
|[Cura](http://wiki.ultimaker.com/Cura)|Yes|Easy|
|[Pronterface](https://github.com/kliment/Printrun)|Yes|Medium|
|[OctoPrint](http://octoprint.org/)|Yes|Easy|

---

## Printing

Once the model is conveted to G-Code, we are ready to print with some kind of printing app!

--- 

## Printing Application
|Name|Open|Ease of Use|
|---|---|---|
|Pronterface|Yes|Medium|
|Cura|Yes|Easy|
|OctoPrint|Yes|Easy|

There are many more apps that can be used!


---

