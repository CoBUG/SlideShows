# What is OpenSMTPD?

1. Program used for sending mail to and from systems.
2. Simple Mail Transfer Protocol (SMTP) daemon.
3. Sendmail compliant interface for sending messages.
4. [RFC 5321](https://tools.ietf.org/html/rfc5321) compliant

---

# Why was another SMTP created?

Dissatisfaction with current implementations of SMTP. Most of them have very
complex configuration.

```
Snippet of sendmail config (M4 macro processor syntax):

1636 ##############################################################
1637 ###  RelayTLS: allow relaying based on TLS authentication
1638 ###
1639 ###     Parameters:
1640 ###             none
1641 ##############################################################
1642 SRelayTLS
1643 # authenticated?
1644 R$*                     $: <?> $&{verify}
1645 R<?> OK                 $: OK           authenticated: continue
1646 R<?> $*                 $@ NO           not authenticated
1647 R$*                     $: $&{cert_issuer}
1648 R$+                     $: $(access CERTISSUER:$1 $)
1649 RRELAY                  $# RELAY
1650 RSUBJECT                $: <@> $&{cert_subject}
1651 R<@> $+                 $: <@> $(access CERTSUBJECT:$1 $)
1652 R<@> RELAY              $# RELAY
1653 R$*                     $: NO
```
Makes eyes bleed!

<span style="font-size: .5em;">Yes those line numbers are accurate (granted there are
comments, but the file length sans comments and white space is ~646 lines!!!)</span>

---

# eye bleach

OpenSMTPD line that does almost* the same thing:

```
lan_addr = "192.168.0.1"
listen on $lan_addr
listen on $lan_addr tls auth
```

<span style="font-size: .5em;">* This is missing the bits to tell OpenSMTPD to
relay the messages, but don't worry, that's just one more line :P</span>

---

# eye bleach breakdown

This line defines a macro that can be later referenced as *$lan_addr*

```
lan_addr = "192.168.0.1"
```

This line tells OpenSMTPD to listen on *$lan_addr* the interface that has the
*192.168.0.1* IP address using the default smtp port (default because nothing
was explicitly set):

```
listen on $lan_addr
```

The second iteration of this line includes the `tls` and `auth` options.

1. `tls` will allow secured connections using STARTTLS on the default port 465.
2. `auth` forces clients to authenticate before they can preform any SMTP
transactions.

---

# to accept or reject

As with any PF style rule configuration, rules are processed in order, from
first to last. This means if you reject something that had a later rule to
accept, it will be lost, as the first rule to match dictates the action taken
for a given message.

### Example ###

```
reject from local for any
accept from local for any relay
```

This would reject any messages originating from the local machine prior to
accepting any.

---

# Matchers

Here is a brief list of things `accept / reject` can match against:

1. `any` : a catch all to match anything
1. `local` : any message originating from the local machine
1. `source *table*` : matches connections originating from a client listed in a
\*table\*
1. `sender *table*` : matches email addresses from \*table\*

## tables ? ###

Tables are essentially lists. They can be a list of IP addresses (say for
matching against `source`), email addresses, domains, user maps.. etc.

---

# !!!!!

Not not not, not not!

All the matchers can be combined with a `!`! A.K.A `not`! This allows for even
greater control in your pattern matching.

Say you have a list of known bad IP addresses:

```
table badies db:/etc/mail/bad_ips.db
accept from ! source <badies> for domain example.org
```

The first line specifies a table called `badies` that contains the list of know
bad ip addresses. The second tells OpenSMTPD to accept any connections *not*
from the `badies` table!

<span style="font-size: .5em;">*This slide and the previous cover a small chunk of the available options. I highly recommend reading [smtpd.conf(5)](https://opensmtpd.org/smtpd.conf.5.html) for a more comprehensive list of options!</span>

---

# my first config

```
listen on lo0

table aliases db:/etc/mail/aliases.db

accept for local alias <aliases> deliver to mbox
accept from local for any relay
```

See if you can figure out what this configuration is allowing and disallowing!

---

# complex examples

```
pki mx1.prism.mx certificate "/etc/ssl/mx1.prism.mx.crt"
pki mx1.prism.mx key "/etc/ssl/private/mx1.prism.mx.key"

pki mail.prism.mx certificate "/etc/ssl/mail.prism.mx.crt"
pki mail.prism.mx key "/etc/ssl/private/mail.prism.mx.key"

queue compression
queue encryption

table userauth sqlite:/etc/mail/prism-sqlite.conf
table userinfo sqlite:/etc/mail/prism-sqlite.conf

listen on lo0 hostname mx1.prism.mx
listen on lo0 port 10028 tag DKIM hostname mx1.prism.mx
listen on mx1.prism.mx tls pki mx1.prism.mx hostname mx1.prism.mx
listen on mail.prism.mx port submission tls pki mail.prism.mx \
	auth <userauth> hostname mail.prism.mx

table aliases { admin = gilles, postmaster = gilles, root = gilles, \
	abuse = gilles }
table sources { 212.83.129.132 }
table names { 212.83.129.132 = mx1.prism.mx }

accept from any for domain prism.mx alias <aliases> \
	userbase <userinfo> deliver to \
	maildmda "/usr/local/bin/prism-mda.py %{user.username}"
accept tagged DKIM for any relay source <sources> helo <names>
accept for any relay via smtp://127.0.0.1:10027
```

---

# More Resources

1. [OpenSMTPD Manual Pages](https://opensmtpd.org/manual.html)
1. [OpenSMTPD Mailing List Archives](http://news.gmane.org/gmane.mail.opensmtpd.general)
1. [OpenSMTPD Source Code](https://github.com/poolpOrg/OpenSMTPD)
1. [Portable Version of OpenSMTPD](https://www.opensmtpd.org/portable.html)
