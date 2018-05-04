---
title: "Ideals"
---

## Simplicity
Varlink aims to be as simple as possible. It is not specifically optimized for anything else but ease-of-use and maintainability. 

## Discoverability
Varlink services describe themselves; with a machine-readable interface definition, and human-readable documentation. 

## Remotability
It should be easy to forward, proxy, redirect varlink interfaces over any connection-oriented transport. Varlink should be free of any side-effects of local APIs. All interactions need to be simple messages on a network, not carrying things like file descriptors or direct references to locally stored files.

## Focus
Varlink is the protocol and definition of interfaces, but it does not define or provide any significant functionality itself.

## Errors
Varlink errors should carry enough information to be consumed by machines and automated rules.

## Testing
Making it easy to run automated tests against varlink interfaces should be considered a necessity when designing interfaces.
