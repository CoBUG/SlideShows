# OpenBSD from Source

![OpenBSD](https://bolddaemon.com/img/banner1.gif)

Why would you want to do this?

---

# Reasons

1. There is an [Errata](http://www.openbsd.org/errata55.html) for your release.
2. You are following -current.
3. Sometimes you just need to compile something.

---

# Rule of Thumb

`man release` will guide you!

It should be considered the single source of truth when it comes to building
OpenBSD releases!

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

Determine if your system is running the multi-processor kernel (MP) or not.
This will determine what config file we use to build.

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

This will print the name of the kernel you built (file you ran `config` on)
plus a revision number, `#N`, this will be incremented every time you build.

---

### Build the rest of your system.

***Taken directly from `release(8)`***

1. Move all your existing object files out of the way and then remove them in
the background:
    ```
    $ cd /usr/obj && mkdir -p .old && sudo mv * .old && \
       sudo rm -rf .old &
    ```
2. Re-build your obj directories:
    ```
    $ cd /usr/src && make obj
    ```
3. Create directories that might be missing:
    ```
    $ cd /usr/src/etc && sudo DESTDIR=/ make distrib-dirs
    ```
4. Begin the build:
    ```
    $ cd /usr/src && make SUDO=sudo build
    ```

Update /etc, /var, and /dev/MAKEDEV, either by hand or using sysmerge(8).

---

# YAY!

You are now running a fully updated system! Depending on the output of
sysmerge, you may want to reboot (if login.conf or friends changed).

---

### Do I have to do this for every *#@&ing machine I own?! ###

---

# No.

---

Remember when you installed OpenBSD, it asked you what `sets` you wanted to
install?

We can cut a `release` that includes the sets (baseXX.tgz.. etc) for our
updated build!

---

## Building a release

Two important shell variables:

1. `DESTDIR` - directory that will contain a full OpenBSD installation. Must be
empty at start of build process.

2. `RELEASEDIR` - directory where the release files are stored.

Make sure you have large enough disks!

---

# ....

Ensure `${DESTDIR}` exists as an empty directory and `${RELEASEDIR}` exists.
`${RELEASEDIR}` need not be empty.  You must be root to create a release:

```
   su
   export DESTDIR=your-destdir; export RELEASEDIR=your-releasedir
   test -d ${DESTDIR} && mv ${DESTDIR} ${DESTDIR}- && \
      rm -rf ${DESTDIR}- &
   mkdir -p ${DESTDIR} ${RELEASEDIR}
```

Make the release and check that the contents of `${DESTDIR}` pretty much
match the contents of the release tarballs:

```
   cd /usr/src/etc && make release
   cd /usr/src/distrib/sets && sh checkflist
   unset RELEASEDIR DESTDIR
```

At this point you have most of an OpenBSD release.

***above taken from `release(8)` - it seriously is your friend!***

---

# All done!

Now we can put the .tgz files on a webserver somewhere and upgrade to our
hearts content!
