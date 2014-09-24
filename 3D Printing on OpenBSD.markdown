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

## Making an apple pie from scratch

Porting 3D printing apps to OpenBSD using the magic that is the ports system!

---

# Gathering requirements

Usually this is a go-until-failure kinda thing, but sometimes you get a break!

OctoPrint has a [list of required python modules](https://github.com/foosel/OctoPrint/blob/master/requirements.txt) included in the github repo!

Searching the ports tree with `make search key=<item>` reveal we are missing 4 of the required libs.

---

# Find source

Fortunately for us the reqs are all on [PyPI](https://pypi.python.org/pypi) and ports has facilities for grabbing stuff from there!

---

## Make a Makefile

Now we build our `Makefile`! You can use `/usr/ports/infrastructure/templates/Makefile.template`, or copy an existing `PyPI` port.

Here is a Makefile for `py-flask-login` (white space removed for slide)

```
COMMENT =		user session management for flask
MODPY_EGG_VERSION =	0.2.11
DISTNAME =	  	Flask-Login-${MODPY_EGG_VERSION}
PKGNAME =		py-${DISTNAME:L}
CATEGORIES =		www
HOMEPAGE = 		https://github.com/maxcountryman/flask-login
# MIT
PERMIT_PACKAGE_CDROM =	Yes
MASTER_SITES =	     	${MASTER_SITE_PYPI:=F/Flask-Login/}
MODULES =    		lang/python
RUN_DEPENDS +=		www/py-flask
MODPY_SETUPTOOLS =	Yes
.include <bsd.port.mk>
```

---

## Important bits

```
MASTER_SITES =	     	${MASTER_SITE_PYPI:=F/Flask-Login/}
```

This bit of magic tells the ports system to grab a tarball from `PyPI`

For a more comprehensive list of available `MASTER_SITES` check out:

`/usr/ports/infrastructure/templates/network.conf.template`

---

## Testing the MASTER_SITE

Once we have our Makefile in place we can test the fetching of our DISTFILE with `make fetch`.

If all goes well you will see a progress bar showing the file you expected downloading.

---

## Building a `distinfo` file

Now we need to create the `SHA256` checksum contained in the `distinfo` file associated with the port.

Ports has your back! `make makesum` will generate your `distinfo` file!

```
qbit@qbit[0]:/usr/ports/www/py-flask-loginλ make makesum
===>  Checking files for py-flask-login-0.2.11
>> Fetch http://pypi.python.org/packages/source/F/Flask-Login/Flask-Login-0.2.11.tar.gz
Flask-Login-0.2.11.tar.gz 100% |***************************************************************************************************************************************| 11099       00:00    
>> Checksum file does not exist
qbit@qbit[0]:/usr/ports/www/py-flask-loginλ cat distinfo                                                                                                                                      
SHA256 (Flask-Login-0.2.11.tar.gz) = g9XxDlxPIU/u1sxBwhLbY6WKFawy5W34FZG/oKXO4+U=
SIZE (Flask-Login-0.2.11.tar.gz) = 11099
```
