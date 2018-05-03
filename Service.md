The [`org.varlink.service`](https://github.com/varlink/libvarlink/blob/master/lib/org.varlink.service.varlink) interface allows clients to retrieve a description of all the interfaces a service implements, and information which describes the service implementation.

```nim
# The Varlink Service Interface is provided by every varlink service. It
# describes the service and the interfaces it implements.
interface org.varlink.service

# Get a list of all the interfaces a service provides and information
# about the implementation.
method GetInfo() -> (
  vendor: string,
  product: string,
  version: string,
  url: string,
  interfaces: string[]
)

# Get the description of an interface that is implemented by this service.
method GetInterfaceDescription(interface: string) -> (description: string)

# The requested interface was not found.
error InterfaceNotFound (interface: string)

# The requested method was not found
error MethodNotFound (method: string)

# The interface defines the requested method, but the service does not
# implement it.
error MethodNotImplemented (method: string)

# One of the passed parameters is invalid.
error InvalidParameter (parameter: string)
```
## Example Information
|Field  |Value                                  |
|-------|---------------------------------------|
|vendor |_Fedora_                               |
|product|_Workstation_                          |
|version|_27_                                   |
|url    |_https://getfedora.org/en/workstation/_|

|Field  |Value                           |
|-------|--------------------------------|
|vendor |_Jon Doe_                       |
|product|_Fidget Service_                |
|version|_0.2_                           |
|url    |_https://example.com/jon/fidget_|
