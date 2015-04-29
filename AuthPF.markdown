# What is AuthPF?

> *authpf* is a user shell for authenticating gateways.

Wut?

---

### No seriously, wut?

  - What is authpf?
  - Why would you want to use it?
  - What does a basic deployment look like?
  - What crazy stuff could you do with it?

---

## What is authpf?

`authpf(8)` is a user shell (think bash or ksh) for authenticating gateways.

It is used to dynamically create pf(4) rules when a user authenticates.  These rules will be removed when the user exits their session.

---

## Why would you want to use it?

1. To grant trusted users more access through a OpenBSD Firewall.
2. To create a "Walled garden" WiFi access point.
3. To restrict access to specific ports on a remote host to a single IP.
4. Can be helpful in troubleshooting firewall connection issues (more on this later).

---

## Configuring pf.conf

All users who successfully authenticate will be given their own pf rules and tables. These need to be anchored in your `pf.conf` with an `anchor` entry:

```
anchor "authpf/*"
```

Once a user authenticates their IP address is added to the `authpf_users` table (must be defined) and a user specific file is parsed to create the pf rules.

---

## Configuring pf.conf (part 2)

Definition of the `authpf_users` table:

```
table <authpf_users persist
```

---

## Configuring custom anchor and table names

If you don't like the default names, you can use `/etc/authpf/authpf.conf` to specify the table / anchor names you do want. Weirdo.

***Side Note:*** This file must exist for authpf to work!

---

## Configuring per-user rules

All user rule templates go in `/etc/authpf/users/$USER`

For example:
```
# cat /etc/authpf/users/abieber
pass from $user_ip
#
```
This rule will allow the user `abieber` to pass all traffic from his IP address!
---

## Configuring per-group rules

Group templates go in .. you guessed it! `/etc/authpf/groups/$GROUP/`

---

## Configuring global rules

Rules can be configured globally with `/etc/authpf/authpf.rules`

**Side Note:** This file must also exist for authpf to work!

---

## Example Global Rule

Shamelessly taken straight from authpf(8):

```
     internal_if="fxp1"
     ipsec_gw="10.2.3.4"

     # rdr ftp for proxying by ftp-proxy(8)
     match in on $internal_if proto tcp from $user_ip to any port 21 \
           rdr-to 127.0.0.1 port 8021

     # allow out ftp, ssh, www and https only, and allow user to negotiate
     # ipsec with the ipsec server.
     pass in log quick on $internal_if proto tcp from $user_ip to any \
           port { 21, 22, 80, 443 }
     pass in quick on $internal_if proto tcp from $user_ip to any \
           port { 21, 22, 80, 443 }
     pass in quick proto udp from $user_ip to $ipsec_gw port = isakmp
     pass in quick proto esp from $user_ip to $ipsec_gw
```

---

## Example Global Rule - Breakdown

  - Built to allow users basic connection to the network.
  - Allows for users to negotiate ipsec and "break free from the chains"
  - But only after they have authenticated with authpf!

---

## Advanced use case

  - Escaping bandwidth throttling.

```
........
queue rootq on $ext_if bandwidth 1000M max 1000M
        queue defq parent rootq bandwidth 1000M default
        queue jerk parent rootq bandwidth 1K max 1K burst 6K for 500ms
........
match proto tcp from !<authpf_users> to any set queue jerk
........
```
---

## Using authpf to troubleshoot firewall issues

Say you have a firewall that is only allowing the following list of outbound traffic:

```
"{ftp, ssh, domain, http, https}"
```

Obviously everything that isn't going over one of these ports will be blocked.

---

## FW Troubleshooting continued..

But sometimes it isn't obvious why something isn't working.

  - Some web pages load content over non-standard ports.
  - Websockets!

A quick method for troubleshooting is to create a user that has all traffic passed without restrictions.

---

## FW Troubleshooting continued..

```
# cat /etc/authpf/users/abieber
pass from $user_ip
#
```

Now when ever abieber authenticates with the gateway, it will create a rule that allows all of his traffic to flow freely!

---

# Warnings

  - Rule order is extremely important!
  - Giving a user unrestricted access might be a bad idea!
    - What if there is some malicious JavaScript that runs on a web page they visit..
      XMLHttpRequest can open a socket to just about any port!
