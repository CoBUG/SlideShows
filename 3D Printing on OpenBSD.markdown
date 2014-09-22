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

## Slicing

What the heck is slicing? Why do we need it?

Have you ever done the "Robot Programming" exercise?

---

## Slicing (cont...)

Slicing is basically cutting a 3D model into commands that the printer turns into physical actions (think Robot Programming.. go forward 3 steps,
 turn left).

Generally slicers produce (G-Code)[https://en.wikipedia.org/wiki/G-code] which is the final format printing apps like OctoPrint use.
---

## Reasons for slicing

  - Controlers on 3D printers are fairly basic, can't run large codebase.
  - 3D objects need to be represented in a linear set if instructions.

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


