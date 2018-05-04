---
title: "Clients"
---

## varlink command line tool
```nim
$ varlink call org.varlink.sysinfo.GetInfo
{
  "parameters": {
    "hostname": "ank",
    "system": {
      "id": "fedora",
      "kernel_version": "4.12.14-300.fc26.x86_64",
      "name": "Fedora",
      "version": "26"
    },
    "virtualization": {
      "name": "none"
    }
  }
}
```

## socat
```nim
$ printf "%s\0" '{"method":"org.varlink.sysinfo.GetInfo"}' | \
    socat - UNIX-CONNECT:/run/org.varlink.sysinfo | jq
{
  "parameters": {
    "hostname": "ank",
    "system": {
      "id": "fedora",
      "kernel_version": "4.12.14-300.fc26.x86_64",
      "name": "Fedora",
      "version": "26"
    },
    "virtualization": {
      "name": "none"
    }
  }
}
```

## Python
```python
$ python
Python 2.7.13 (default, Sep  5 2017, 08:53:59) 
Type "help", "copyright", "credits" or "license" for more information.
>>> import json
>>> import socket
>>> s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
>>> s.connect('/run/org.varlink.sysinfo')
>>> call = { 'method': 'org.varlink.sysinfo.GetInfo' }
>>> s.send((json.dumps(call) + '\0').encode('utf-8'))
>>> reply = json.loads(s.recv(1000)[:-1])
>>> print(reply)
{u'parameters': {u'hostname': u'ank', u'system': {u'kernel_version': u'4.12.14-300.fc26.x86_64', u'version': u'26', u'id': u'fedora', u'name': u'Fedora'}, u'virtualization': {u'name': u'none'}}}
```