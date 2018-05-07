---
title: "Adding varlink to DNF"
toc: true
---

This page shows, how to easily add a varlink interface to existing software.

## Prerequisites
Checkout the (dnf git repo with branch ```varlink```)[https://github.com/haraldh/dnf]:

```bash
$ git clone -b varlink https://github.com/haraldh/dnf.git
$ cd dnf
$ cmake . -DPYTHON_DESIRED:str=3 -DWITH_MAN=0 -DPYTHON_EXECUTABLE=/usr/bin/python3
$ make

```
For Fedora we also need 
```bash
$ sudo dnf install --enablerepo rawhide python3-varlink libvarlink-util
```

## Interface definition
First, the interface definition has to be specified. With compatibility in mind, we define a general purpose package manager interface: [com.redhat.packages.varlink](https://github.com/haraldh/dnf/blob/varlink/dnf/varlink/com.redhat.packages.varlink).

## Implementation
The implementation is done in (main.py)[https://github.com/haraldh/dnf/blob/varlink/dnf/varlink/main.py]

First a ```varlink.Service``` object is created:
```python
service = varlink.Service(
    vendor='Red Hat',
    product='Packages',
    version='1',
    interface_dir=os.path.dirname(__file__),
    namespaced=True
)
```

The ```namespaced=True``` indicates, that the methods are called with arguments of type ```SimpleNamespace``` and not ```dict```, so that we can access the subelements with ```parm.foo``` instead of ```parm["foo"]```.

The ```DnfVarlinkService``` class is tagged with the ```@service.interface('com.redhat.packages')``` annotator, to indicate, that it is the interface provider for the varlink service ```com.redhat.packages```. The ```varlink.Service``` will load the interface definition file ```com.redhat.packages.varlink``` from the directory specified with ```interface_dir```.

For our showcase, we implement only the ```List``` method. First we create the ```search_pattern``` according to the ```packages``` parameter. After querying the packages with ```base._do_package_lists()```, the returned lists have to be processed to fit the ```Package``` type specified in the varlink ```com.redhat.packages``` interface.

The ```varlink.ThreadingServer``` class, which is a subclass of [socketserver.BaseServer](https://docs.python.org/3/library/socketserver.html) is used to serve from the URL, which is passed in ```argv[1]``` (for dnf ```args[0]```). ```varlink.ThreadingServer```, ```varlink.ForkingServer``` and ```varlink.Server``` parse the varlink URI address, check for socket activation (LISTEN_FD) and bind a socket for us.

```main()``` is called from the [dnf-varlink](https://github.com/haraldh/dnf/blob/varlink/bin/dnf-varlink.in) cli utility following ```dnf-automatic``` and ```dnf-cli``` in the [bin](https://github.com/haraldh/dnf/tree/varlink/bin) directory.

## Running
Now we can talk to dnf with varlink. We use the "exec:" mechanism, which will start the ```dnf-varlink-3``` client executable directly in the user context with all the user's permissions.
```javascript
$ varlink call exec:./bin/dnf-varlink-3/com.redhat.packages.List '{"packages": [{"name":"libvarlink", "version": {"architecture": "x86_64" }}, {"name": "python3-varlink"}]}'
Last metadata expiration check: 0:04:35 ago on Mi 21 Feb 2018 10:02:42 CET.
{
  "packages": [
    {
      "available": false,
      "installed": true,
      "name": "libvarlink",
      "version": {
        "architecture": "x86_64",
        "release": "2.fc28",
        "version": "1"
      }
    },
    {
      "available": true,
      "installed": true,
      "name": "python3-varlink",
      "version": {
        "architecture": "noarch",
        "release": "1.git.137.51a669b.fc28",
        "version": "12"
      }
    },
    {
      "available": true,
      "installed": false,
      "name": "libvarlink",
      "version": {
        "architecture": "x86_64",
        "release": "1.git.224.bd4cd60.fc28",
        "version": "2"
      }
    }
  ]
}
```

The ```dnf-varlink-3``` executable can also be started in server mode
```bash
$ ./bin/dnf-varlink-3 --varlink=unix:@com.redhat.packages
```

And a client can talk to it via the unix socket
```javascript
$ varlink call unix:@com.redhat.packages/com.redhat.packages.List '{"packages": [{"name":"libvarlink", "version": {"architecture": "x86_64" }}, {"name": "python3-varlink"}]}'
{
  "packages": [
    {
      "available": false,
      "installed": true,
      "name": "libvarlink",
      "version": {
        "architecture": "x86_64",
        "release": "2.fc28",
        "version": "1"
      }
    },
    {
      "available": true,
      "installed": true,
      "name": "python3-varlink",
      "version": {
        "architecture": "noarch",
        "release": "1.git.137.51a669b.fc28",
        "version": "12"
      }
    },
    {
      "available": true,
      "installed": false,
      "name": "libvarlink",
      "version": {
        "architecture": "x86_64",
        "release": "1.git.224.bd4cd60.fc28",
        "version": "2"
      }
    }
  ]
}
```

Other transport mechanisms can be specified via the [varlink address](/#address). ```varlink.SimpleServer``` supports ```exec:```, ```unix:``` and ```tcp:```.
