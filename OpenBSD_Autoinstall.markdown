# OpenBSD 
# Autoinstall

![OpenBSD](https://deftly.net/banner1.gif)

---

## Agenda


  - what is it?
  - why anyone would use it?
  - deploy a basic rig
  - demonstrate a complicated rig

---

## What is it?

`autoinstall(8)` is an install method that was added in OpenBSD 5.5.

The text-based installation procedure can be fully automated through the use of
a preseeded configuration file containing answers to the questions.

---

## Why would you use it?

There are plenty of reasons why someone would want to use this feature,
including but not limited to:

1. elastic environments 
2. basis for a hosting product 
3. easier installation on isolated/remote computers 
4. easier regression testing 
5. it is cool

---

## What does it look like?

Everything one needs to get going is already installed under OpenBSD. There is
no need to install any packages. The core components of an autoinstall rig are:

  - **[dhcp server]** 
    - responds to dhcp requests from netboot/pxe clients
    - returns `next-server` and `filename` to pxe client, telling it what to grab next
  - **[tftp server]** 
    - serves `/bsd.rd` kernel
    - serves `/pxeboot` (as `/auto_install`)
  - **[web server]** 
    - serves installation sets (like you would see in a public mirror)

---

## Overview of steps

For a high level overview, here are the things we will do in this tutorial:

  * configure dhcpd 
  * configure tftpd 
  * configure a webserver 
  * synchronize openbsd installation media into the webroot 
  * build an install.conf file

---

## Configuring DHCPD: Part 1

I run dhcpd from my router, however, it can be run from anywhere as long as you
have control of the broadcast domain. Add a stanza similar to this into your
`/etc/dhcpd.conf`:

<code console>
    # grep cobug /etc/dhcpd.conf
    subnet 192.168.10.0 netmask 255.255.255.0 { # cobug example 
        range 192.168.10.150 192.168.10.250;    # cobug example 
        option routers 192.168.10.1;            # cobug example 
        next-server 192.168.10.1;               # cobug example 
        filename "auto_install";                # cobug example 
    }                                           # cobug example
</code>

---

## Configuring DHCPD: Part 2

Make sure it runs on boot 

<code console>
    # grep 'dhcpd_flags' /etc/rc.conf.local || \
      echo 'dhcpd_flags=""' >> /etc/rc.conf.local
</code>

start dhcpd

<code console>
    # /etc/rc.d/dhcpd restart
    dhcpd(ok) 
    dhcpd(ok)
</code>

---

## Configuring tftpd: Part 1

We will point tftpd to use the webroot instead of the default.  We want to do
this because we will also be serving out the install media from there, which
already contains the `pxeboot` and `bsd.rd` files we need. 

Also because it lets us keep all the autoinstall files together in one place.

---

## Configuring tftpd: Part 2

enable it on boot and point it to the webroot:

<code console>
    # grep 'tftpd_flags' /etc/rc.conf.local || \
      echo 'tftpd_flags="/var/www/htdocs/"' >> /etc/rc.conf.local
</code>

start it

<code console>
    # /etc/rc.d/tftpd restart
    tftpd(ok)
    tftpd(ok)
</code>

---

## Configuring the webserver: Part 1

Bandwidth is not free, so we will start a webserver and use it to serve the
installation media we download. We will be making use of the webserver in the
latter half of this presentation.

---

## Configuring the webserver: Part 2

enable it on boot 

<code console>
    # grep 'nginx_flags=""' /etc/rc.conf.local || \
      echo 'nginx_flags=""' >> rc.conf.local 
</code>

start it now

<code console>
    # /etc/rc.d/nginx start
    nginx(ok)
</code>

---

## Getting the installation media: Part 1

You need to download all of the files you see in a url like
`http://ftp.usa.openbsd.org/pub/OpenBSD/5.5/amd64/` (assuming you're installing amd64)

OpenBSD does not come with wget, but it does come with an http-compatible `ftp`
binary, which we will use like so:

<code console>
    # URL=http://ftp.usa.openbsd.org/pub/OpenBSD/5.5/amd64/
    # mkdir -p /var/www/htdocs/5.5/
    # cd /var/www/htdocs/5.5
    # ftp -Vio- ${URL}/SHA256 | \
      sed -e 's/(//g;s/)//g' | \
      awk '{print $2}' | \
      while read L; do ftp ${URL}/${L}; done
    # ftp ${URL}/SHA256{,.sig}
</code>

---

## Getting the installation media: Part 2

Now that you have the installation media, symlink the pxeboot file in `5.5/` to a
file called `auto_install`

<code console>
    # pwd
    /var/www/htdocs
    # ln -s 5.5/pxeboot ./auto_install
    # ls -l
    total 12
    drwxr-xr-x  2 root  daemon  1024 May 27 16:41 5.5
    lrwxr-xr-x  1 root  daemon    11 May 27 16:48 auto_install -> 5.5/pxeboot
</code>

We named it auto_install for two reasons:

  - we set `filename "auto_install";` in the `dhcpd.conf`
  - we wish to install, not upgrade or boot to the interactive installer. By
    naming it `auto_install` the installer knows it must initiate the
autoinstall process (as opposed to the autoupgrade or interactive installer)

---

## Getting the installation media: part 3

Now finally, set the `bsd` kernel in the tftp/web root so we can actually boot it:

<code console>
    # pwd
    /var/www/htdocs
    # ln -s 5.5/bsd.rd bsd
    # ls -l
    total 12
    drwxr-xr-x  2 root  daemon  1024 May 27 16:41 5.5
    lrwxr-xr-x  1 root  daemon    10 May 27 16:58 bsd -> 5.5/bsd.rd
    lrwxr-xr-x  1 root  daemon    11 May 27 16:48 auto_install -> 5.5/pxeboot
</code>

---

## Building an install.conf: Part 1

Now that everything is coming together, we have one last step to do: build an
openbsd `install.conf` file. 

As you probably know, installing openbsd involves answering a series of
questions - most of which you go with the default on.  However, in some cases
they are not sufficient (root password comes to mind). 

In the event that you would like to change any of the default answers, you
simply create an `install.conf` in your tftp root with answers to the questions
asked by the installer.

---

## Building an install.conf: Part 2

### Example install.conf

    system hostname = unconfigured
    password for root account = 2insecure4me
    network interfaces = em0
    IPv4 address for em0 = dhcp
    Location of sets? = http
    server? = 192.168.10.1
    server directory? = 5.5/

---

## Building an install.conf: Part 3

Make sure this `install.conf` file is in the webroot - it has to be
available at:

    http://<next-server>/install.conf

In my lab, the setup looks something like this:

<code console>
    # pwd
    /var/www/htdocs
    # ls -l
    total 16
    drwxr-xr-x  2 root  daemon  1024 May 27 16:41 5.5
    lrwxr-xr-x  1 root  daemon    11 May 27 16:59 auto_install -> 5.5/pxeboot
    lrwxr-xr-x  1 root  daemon    10 May 27 16:58 bsd -> 5.5/bsd.rd
    -rw-r--r--  1 root  daemon   365 May 27 17:41 install.conf
</code>

---

## Building an install.conf: Part 3 (cont'd)

<code console>
    # cat install.conf
    system hostname = unconfigured
    password for root account = 2insecure4me
    Do you expect to run the X Window System? = yes
    Change the default console to com0? = yes
    What timezone are you in? = US/Mountain
    Location of sets? = http
    server? = 192.168.10.1
    server directory? = 5.5/
</code>

---

## Example build 
## and basic autoinstall

[![Screencast of install](http://blog.niko.im/sc1.png)](https://asciinema.org/a/9931?speed=5&size=medium&autoplay=1)

---

## How could we 
## build on this?

  * use `site${release}.tgz` to bootstrap your favorite software (CF Management comes to mind)
  * monitor multicast responses in order to bootstrap new clients
  * write an API-driven, dynamic `install.conf` generator (so that all install-answers are custom on a per-client/MAC basis)

---

## Complicated example

In my lab, I have a similar setup to the one demonstrated earlier , however, I use `httpd`, `cgi-perl`,
`mod_rewrite` and `sqlite3` to serve different `install.conf`s that are built dynamically. This enables me to create and configure openbsd systems with a single command.

---

## Complicated Example (part 2)

Suppose I wanted to create a new VM called `hobosandwiches` with a
different site.tgz, and root key from the rest of my environment.

  * run shellscript with arguments defining the pieces of the virtual machine

<code console>
    # ./create_vm.sh --name hobosandwiches \
        --sitefile ./sitefiles/hobos_site.tgz \
        --rootkey "$(cat keys/hobo.pub)"
</code>

---

## Complicated Example (part 3)

  * When this script executes, it will run a SQL INSERT into a sqlite database, like so

<code console>

    INSERT INTO vms (name,sitefile,rootkey) 
      VALUES(
        'hobosandwiches',
        './sitefiles/hobos_site.tgz',
        'ssh-rsafooooooooooooooooooooooo');
</code>

---

## Complicated Example (part 4)

  * Then, the script will go to a chosen hypervisor and actually create the VM.
  * During the VM creation phase, a child will fork off and watch the output for the moment a MAC address is assigned
  * When the MAC address is assigned, it is saved and the record we just added to the DB is updated

    `UPDATE vms SET mac='mac address' WHERE name='hobosandwiches';`

---

## Complicated Example (part 5)

  * In the meantime, the VM is created and boots
  * When the VM boots, it checks `http://next-server/MACaddress-install.conf`
  * apache is configured to rewrite that URL to `install.conf?mac=<mac>`

    `RewriteRule ^/?(.*)-install.conf$ /install.conf?mac=$1 [L]`

  * Apache is also configured to serve install.conf as a perl script

    `AddHandler cgi-script .conf`

---

## Complicated example (part 6)

Finally, when `install.sh` runs on the client and looks for `http://next-server/MACaddress-install.conf`, a customized response is delivered.

---

## Questions / Comments
### Flames / Presents / Cakes
