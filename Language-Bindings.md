---
title: "Language Bindings"
---

## Existing language bindings
* [C](https://github.com/varlink/libvarlink)
* [Go](https://github.com/varlink/go)
* [Python](https://github.com/varlink/python)
* [Rust](https://github.com/varlink/rust)

## OS Support

T
: [TCP](/#address) sockets

U
: [UNIX](/#address) domain sockets

A
: [Activation](/#activation) via file descriptors

B
: [Bridge](/#bridge) mode in CLI tool

b
: [Bridge](/#bridge) mode in client connection

|            | Linux | MacOS | Win10 |
| -----------|:------|:------|:------|
| C          | TUABb |       |       |
| Go         | TUABb | TUABb | Tb    |
| Python     | TUABb | TUABb | Tb    |
| Rust       | TUABb | TUABb | TUBb  |

## How to test new language bindings

Checkout the python varlink git repo:
```shell
$ git clone https://github.com/varlink/python.git
$ cd python
$ cat tests/org.varlink.certification.varlink
```
Write a varlink client for org.varlink.certification calling the methods:

```go
Start(), Test01(), Test02(…), …, Testxx(…), End()
```

The return value of the previous call should be the argument of the next call.
See the example clients in [python](https://github.com/varlink/python/blob/master/varlink/tests/test_certification.py) or [rust](https://github.com/varlink/rust/blob/master/varlink-certification/src/main.rs#L73-L138).

Then you test your client against the python varlink certification server:

```shell
$ python -m varlink.tests.test_certification --varlink=tcp:127.0.0.1:12345
```

which now listens on 127.0.0.1:12345.

Next you write a varlink server providing the org.varlink.certification service like the python service.
Now run your client against it and run well known clients like python or rust
from https://github.com/varlink/ against your server. If all works out, then
your new language bindings should be varlink certified.

```markdown
[<img src="/images/varlink-certified.png" alt="varlink-certified" width="240" height="73">](https://varlink.org/Language-Bindings)
```

[<img src="/images/varlink-certified.png" alt="varlink-certified" width="240" height="73">](https://varlink.org/Language-Bindings)
