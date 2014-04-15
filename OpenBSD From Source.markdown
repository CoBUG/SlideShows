# OpenBSD from Source

![OpenBSD](https://deftly.net/banner1.gif)

Why would you want to do this?

---

# Reasons

1. There is an [Errata](http://www.openbsd.org/errata55.html) for your release.
2. You are following -current.
3. Sometimes you just need to compile something.

---

# Rule of Thumb

`man release` will guide you!

---

# Basic steps

1. Get / Update your source.
2. Build and install your kernel.
3. Reboot into your new kernel.
4. Build your full system. (Optional depending on what you are doing).
5. Cut some sets for distribution of your freshly updated system. (Also optional)

---

# Preliminary configuration of CVS:

Typical `.cvsrc`:

```
# $OpenBSD: dot.cvsrc,v 1.1 2013/03/31 21:46:53 espie Exp $
#
diff -uNp
update -P
checkout -P
```

With aditional line:

```
cvs -danoncvs@anoncvs3.usa.openbsd.org:/cvs
```

*[anoncvs3.usa.openbsd.org](http://www.openbsd.org/ftp.html) and [ftp3.usa.openbsd.org](http://www.openbsd.org/ftp.html) are hosted in Colorado!*

---

# Getting the source using cvs:

1. Source for a specific release (where TAG is something like OPENBSD_5_5).

    `cd /usr; cvs checkout -P -rTAG src`

2. Source for -current

    `cd /usr; cvs checkout -P src`

---

# Getting the source using tar balls:

*** - This does not get you the latest Errata for your release! - ***

```
cd /usr/src
# Get your base source
ftp http://ftp3.usa.openbsd.org/pub/OpenBSD/5.4/src.tar.gz
# Get the kernel source
ftp http://ftp3.usa.openbsd.org/pub/OpenBSD/5.4/sys.tar.gz
tar -zxvf src.tar.gz
tar -zxvf sys.tar.gz
```

Next pull in the latest Errata for your release (5.4 in this case)!

```
cvs up -rOPENBSD_5_4
```

---

# Now we are ready to build!

---

# First we build our kernel.

```
cd /usr/src/sys/arch/$(uname -m)/conf
```

Determine if your system is running the multi-processor kernel (MP) or not. This will determine what config file we use to build.

```
uname -a
```

Configure the kernel:

```
config GENERIC  # Use GENERIC.MP if you system is a multi-processor
```

---
# ....

Build the kernel with:

```
cd ../compile/GENERIC/ && make clean && make && sudo make install
```

Reboot into new kernel!

```
sudo reboot
```

Once your machine is back up and running, verify the new kernel:

```
uname -v
```

This will print the name of the kernel you built (file you ran `config` on) plus a revision number, `#N`, this will be incremented every time you build.

---


