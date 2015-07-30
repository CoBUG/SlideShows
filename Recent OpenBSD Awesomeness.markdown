# Recent Awesome - 2015-07-30

- The Jump
- Taming the Beast
- Do as I say, not as I do
- Hell - Why is it so cold down here?

---

# The Jump

---

Now in `mandoc` one can add "tags" to pages, allowing users to hop around to various locations.

---

Why not just use `/`?

---

What does it work with?

- command line options
- internal commands
- environment variables
- command modifiers  

---
# Taming the Beast
---

From the man page:

`The current process is forced into a restricted-service operating mode.
     A few subsets are available, roughly described as computation, memory
     management, read-write operations on file descriptors, opening of files,
     networking.  In general, these modes were selected by studying the
     operation of many programs using libc and other such interfaces.`

---

- Restrict system operations
- Simple to add to existing applications
- Attempts to access restricted syscalls result in `SIGKILL`

---

### Operations that can be restricted:

- computation
- memory management
- read-write operations on file descriptors
- opening of files
- networking

---

### Simple to add

```
Index: bin/cat/cat.c
===================================================================
RCS file: /cvs/src/bin/cat/cat.c,v
retrieving revision 1.21
diff -u -p -u -r1.21 cat.c
--- bin/cat/cat.c	16 Jan 2015 06:39:28 -0000	1.21
+++ bin/cat/cat.c	24 May 2015 01:03:25 -0000
@@ -35,6 +35,7 @@
 
 #include <sys/types.h>
 #include <sys/stat.h>
+#include <sys/tame.h>
 
 #include <ctype.h>
 #include <err.h>
@@ -65,6 +66,8 @@ main(int argc, char *argv[])
 	int ch;
 
 	setlocale(LC_ALL, "");
+
+	tame(TAME_STDIO | TAME_RPATH);
 
 	while ((ch = getopt(argc, argv, "benstuv")) != -1)
 		switch (ch) {
```
---

# Do as I say, not as I do

---

### doas

- `sudo` replacement
- much simpler implementation
- familiar syntax
- NIH syndrome?

---

### doas
Clocking in at a slim 660ish LOC, `doas` is a much slimmed down priv esc app.

---

### Familiar syntax

```
permit jimbob cmd /sbin/reboot
permit nopass keepenv { HTTP_proxy } :wheel
```
---

### NIH Syndrome

- Could be, but not likely.
- Have you seen the things `sudo` can do?

---

# Hell - Why is it so cold down here?

---

Microsoft has donated at least 25K to the OpenBSD Foundation \o/

---

Oracle is adding OpenSSH and PF to Solaris!
