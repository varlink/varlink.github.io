---
title: ""
toc: true
---

Varlink is an interface description format and protocol that aims to make services accessible to both humans and machines in the simplest feasible way.

A varlink interface combines the classic UNIX [command line options](https://en.wikipedia.org/wiki/Command-line_interface), [STDIN/OUT/ERROR](https://en.wikipedia.org/wiki/Standard_streams) text formats, [man pages](https://en.wikipedia.org/wiki/Man_page), service metadata and provides the equivalent over a single file descriptor, a.k.a. ["FD3"](#activation).

Varlink is plain-text, type-safe, discoverable, self-documenting, remotable, testable, easy to debug. Varlink is accessible from any programming environment. See the [Ideals](Ideals) page for more. And everybody likes [Screenshots](Screenshots).

## Interface
A varlink interface has a reverse-domain name and specifies which methods the interface implements. Each method has named and typed input and output parameters. Complex types can be aliased with the `type` keyword to allow reusing them and to make method signatures easier to read. The interface also specifies the errors that may be returned from its method calls.

Everything can be documented by adding a comment immediately before it. The documentation is provided to clients as structured data on a well-known service interface.

See the [Interface Definition](Interface-Definition) about the varlink syntax and how to parse an interface file.

```nim
# Interface to jump a spacecraft to another point in space.
# The FTL Drive is the propulsion system to achieve
# faster-than-light travel through space. A ship making a
# properly calculated jump can arrive safely in planetary
# orbit, or alongside other ships or spaceborne objects.
interface org.example.ftl

# The current state of the FTL drive and the amount of
# fuel available to jump.
type DriveCondition (
  state: (idle, spooling, busy),
  tylium_level: int
)

# Speed, trajectory and jump duration is calculated prior
# to activating the FTL drive.
type DriveConfiguration (
  speed: int,
  trajectory: int,
  duration: int
)

# The galactic coordinates use the Sun as the origin.
# Galactic longitude is measured with primary direction
# from the Sun to the center of the galaxy in the galactic
# plane, while the galactic latitude measures the angle
# of the object above the galactic plane.
type Coordinate (
  longitude: float,
  latitude: float,
  distance: int
)

# Monitor the drive. The method will reply with an update
# whenever the drive's state changes
method Monitor() -> (condition: DriveCondition)

# Calculate the drive's jump parameters from the current
# position to the target position in the galaxy
method CalculateConfiguration(
  current: Coordinate,
  target: Coordinate
) -> (configuration: DriveConfiguration)

# Jump to the calculated point in space
method Jump(configuration: DriveConfiguration) -> ()

# There is not enough tylium to jump with the given
# parameters
error NotEnoughEnergy ()

# The supplied parameters are outside the supported range
error ParameterOutOfRange (field: string)
```

## Protocol
All messages are encoded as JSON objects and terminated with a single `NUL` byte.

For simplicity reasons and to minimize the state a client needs to track; for every connection, the service sends its responses in the same order that the requests were received. Requests can be [pipelined](https://en.wikipedia.org/wiki/HTTP_pipelining) but not multiplexed; a pending method reply occupies the connection.

The common case is a simple method call with a single reply. To support _monitoring calls_, _subscriptions_, _chunked data_, _streaming_, calls may carry instructions for the server to not reply, or to reply multiple times to a single method call. See the [Method Call](Method-Call) page for a detailed description.

In common programming languages, varlink clients do not require complex modules or libraries, already existing JSON and socket communication facilities are used to integrate natively into the programming language's object model.

Requests specify the fully-qualified `method` that should be called, along with its input parameters:
```javascript
{
  "method": "org.example.ftl.CalculateConfiguration",
  "parameters": {
    "current": {
      "longitude": "27.13",
      "latitude": "-12.4",
      "distance": "48732498234"
    },
    "target": {
      "longitude": "-48.7",
      "latitude": "12.9",
      "distance": "354667658787"
    }
  }
}
```

A service replies with an object that contains the output `parameters`:
```javascript
{
  "parameters": {
    "configuration": {
      "speed": "32434234",
      "trajectory": "686787",
      "duration": "13256445"
    }
  }
}
```

Errors contain the fully-qualified error and optional `parameters` as specified and documented in the varlink interface file:
```javascript
{
  "error": "org.example.ftl.ParameterOutOfRange",
  "parameters": { 
    "field": "current.distance"
  }
}
```

## Service
Every varlink service offers the `org.varlink.service` interface, which describes all interfaces the service provides and provides information about the service implementation itself. See the [Service](Service) page for details.

## Address
Varlink services are expressed in URI form:

|Type                          |Example                         |Comment                                  |
|------------------------------|--------------------------------|-----------------------------------------|
|TCP                           |_tcp:127.0.0.1:12345_           |hostname/IP address and port             |
|UNIX socket                   |_unix:/run/org.example.ftl_     |optional access `;mode=0666` parameter   |
|UNIX abstract namespace socket|_unix:@org.example.ftl_         |                                         |
|device node                   |_device:/dev/org.kernel.example_|                                         |

See the [transport screenshot](Screenshots#transports) for examples.

## Activation
A listen socket a.k.a "FD3" might be passed to a varlink service at startup.

|Environment   |Example                       |Comment                                                          |
|--------------|------------------------------|-----------------------------------------------------------------|
|LISTEN_FDS    |_LISTEN_FDS=2_                |Number of file descriptors passed to the service. It always starts at 3. If more than one file descriptor is passed, _LISTEN_FDNAMES_ identifies the _varlink_ file descriptor.|
|LISTEN_PID    |_LISTEN_PID=4711_             |The process id of the started service. This should match the PID of the started service, otherwise everything should be ignored.|
|LISTEN_FDNAMES|_LISTEN_FDNAMES=other:varlink_|A colon separated list of names of the passed file descriptors.  The varlink file descriptor should be named _varlink_.|

The service activator should pass the command line option _--varlink=[ADDRESS](#address)_ to the service, to make the listening address and possible parameters known to the service. It also shows the listening address in _ps_, which helps to debug a running service.

See an example implementation of a [service activator](https://github.com/cherry-pick/com.redhat.resolver/blob/master/src/service.c#L112).

## Resolver
Public varlink interfaces are registered system-wide by their well-known address, by default _/run/org.varlink.resolver_. The resolver translates a given varlink interface to the service address which provides this interface.

Multiple services can implement and offer the same interface, but only one of the services is registered with the resolver. The set of registered interfaces becomes the globally visible system interface, its actual configuration is usually defined by the operating system and not managed by the services themselves.

See the [`org.varlink.resolver`](https://github.com/cherry-pick/com.redhat.resolver/blob/master/src/org.varlink.resolver.varlink) interface for details.

## Bridge
The [varlink command line tool](Clients#varlink-command-line-tool) supports a `bridge` mode to bridge a single connection to the resolver and its registered services. It intercepts the calls to the `org.varlink.service` interface and replies with the information the resolver supplies.

If the bridge is used over SSH, all the interfaces of the locally running services appear to the remote ssh client as if they were implemented by the bridge.

See a [screenshot](Screenshots#remote-host) as an example.
