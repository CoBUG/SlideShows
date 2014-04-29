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

---

# eye bleach

OpenSMTPD line that does almost* the same thing:

```
lan_addr = "192.168.0.1"
listen on $lan_addr
listen on $lan_addr tls auth
```

<span style="font-size: .5em;">* This is missing the bits to tell OpenSMTPD to
relay the messages.</span>

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

1. *tls* will allow secured connections using STARTTLS on the default port 465.
2. *auth* forces clients to authenticate before they can preform any SMTP
transactions.

---


