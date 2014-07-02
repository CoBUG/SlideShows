# What is PF?

PF or Packet Filter, is a filtering mechanism which lives
in the kernel, has a userland interface (`/dev/pf`) and tools (`pfctl`)
for manipulating rules, table entries and gathering statistics.

---

# What can you do with PF?

1. Filtering based on any number of connection or packet properties.
2. Redirection (send this packet to this host).
3. Bandwidth control (via queues)
4. Direct traffic to external applications.

.. Much more!

---

## Basic Concepts

### Macros

Can be thought of as variables, but they allow you to specify lists of 
items as well.

```
  ext_if = "em0"
  all_ifs = "{" $ext_if lo0 "}"
```

### Tables

Are collections of addresses or networks.

```
  table <goodhosts> { 10.0.1.2, 10.0.1.3 }
  table <badhosts> file "/etc/pf/bads.list"
```

### Rules

Define what should be done with packets that match a specific criteria.

```
  block on $ext_if from <badhosts>
  pass in on $ext_if proto tcp from any to $ext_if port www
```

---

## ...Basic Concepts

`pfctl` a command run to enable / disable pf, load config changes, change states and view statistics.

**Enabling:**
```
pfctl -e
```

**Disabling:**
```
pfctl -d
```

**Showing All Stats:**
```
pfctl -s all
```

---

## ...Basic Concepts

**Reload config:**
```
pfctl -f /etc/pf.conf	# you can use -n to do a syntax check
```

**Modify State:**
```
pfctl -k 0.0.0.0/0 -k host1 # kill all states going to host1
```

---

# Filtering

Using the basic syntax listed in the previous slide, we can define some simple filtering rules

```
# do not filter any packets on lo
set skip on lo

# block packets and return TCP RST for TCP and ICMP UNREACHABLE
# for other packets.
block return

# pass packets
pass

# send TCP RST / ICMP UNREACHABLE for packets that hit
# interfaces that are NOT lo0 and are between port 6000 and 6010
block return in on ! lo0 proto tcp to port 6000:6010

```

---

# ...Filtering Example

I call this config the "Go away". This is probably the most minimal ruleset you can have.

It blocks all inbound connections while allowing and keeping state* for any outbound connections.

```
block in all
pass out all
```

*Note since OpenBSD 4.1, state is implicit. Prior to 4.1 your `pass` line would look like:
```
pass out all keep state
```

---

# ...Complex Filtering Example

```
pass in on $ext_if proto tcp from any to $ext_if port ssh rdr-to \
$ssh_host port ssh flags S/SFRA synproxy state (max 5, \
source-track rule, max-src-states 5, max-src-nodes 5, \
max-src-conn-rate  5/60 overload <tmpblock> flush global )
```

This lengthy example is doing a lot of things, but basically, when someone connects to `$ext_if`
on port `22` (ssh) PF redirects the traffic to `$ssh_host` assuming the subsequent
rulesets have not been hit.

---

### Complex rule breakdown

* `flags S/SFRA` - this rule applies to (S)YN TCP packets that have flags in (S)YN, (F)IN, (R)ST and (A)CK and is there to prevent ACK storms.
* `synproxy state` - this rule causes pf(4) to complete the tcp handshake process and forward packets between endpoints. This is used to prevent SYN floods.
* `max 5` - Limits concurrent states the rule can create to 5.
* `source-track rule` - tells PF to track the number of states per source IP. The number allowed will be defined by `max-src-states` and `max-src-nodes`
* `max-src-{states,nodes} 5` tells our `source-track rule` directive to limit the state table entries to 5.

---

### ...Complex rule breakdown

* `max-src-conn-rate 5/60` - Limit the rate of new connections to N/Seconds (5 connections per 60 seconds)
* `<tmpblock>` - the table where connections that hit our limits will be added
* `flush global` kills states for the hosts that exceed the matching rules.

## What does that all mean?!

The entire rule is essentially limiting ssh connections to 5, and temporarily blocking any connections that try to make more than 5 connections in 60 seconds.

---

# More Info

### Awesome Books
* [Book of PF 3rd Edition](http://www.nostarch.com/pf3) - currently only available as early access.
* [Book of PF 2nd Edition](http://www.nostarch.com/pf.htm) - ebook only due to high demand!

### Man pages
* [pf(4)](http://www.openbsd.org/cgi-bin/man.cgi?query=pf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)
* [pfctl(8)](http://www.openbsd.org/cgi-bin/man.cgi?query=pfctl&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)
* [pf.conf(5)](http://www.openbsd.org/cgi-bin/man.cgi?query=pf.conf&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)
