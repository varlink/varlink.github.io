---
title: "Screenshots"
toc: true
---

## Command Line
The varlink command line tool with TAB-completion of locally registered interfaces.

[![commandline](screenshots/commandline.png)](screenshots/commandine.png)

## Connections
The command line tool calling a method which will be locally resolved, or call a UNIX-socket directly,
or execute (socket activation protocol) a binary and connect to it, or connect to a remote host with SSH
and run a varlink bridge on the remote host.

[![connections](screenshots/connections.png)](screenshots/connections.png)

## Remote Host
Running a bridge on the remote system. All local interfaces on the remote host are accessible to e.g. 
a SSH client. The `varlink` command line tool will pass its connection through the binary executed 
with the `--bridge=` option, in this case `ssh`. The bridge executed on the remote side will connect 
to the local services available on the remote machine. It the option `--connect=` is given to the 
remote bridge command, the bridge will connect directly to a known remote address instead of trying to
resolve the address with the local resolver.

[![remote-host](screenshots/remote-host.png)](screenshots/remote-host.png)

## Local system
The local activator starting services on-demand and and resolver providing the addresses of requested interfaces.

[![local-system](screenshots/local-system.png)](screenshots/local-system.png)

## Kernel Module
This proof-of-concept kernel module exports a varlink service over a device node, implementing system
information and USB monitoring interfaces.

[![kernel](screenshots/kernel.png)](screenshots/kernel.png)
