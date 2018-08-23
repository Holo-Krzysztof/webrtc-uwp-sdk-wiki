
# Overview

For the WebRTC UWP project, we've decided to use [IDLs](https://en.wikipedia.org/wiki/IDL_specification_language) (Interface Definition Language) to auto generate the wrapper code we need to expose the internal C++ APIs in other language bindings we wish to expose to developers externally. IDL defines a set of objects, structures, lists, maps, sets, dictionaries, properties, exceptions, object events, and asynchronous promises in a "header" only style simple language

## Rational

Specifically for the WebRTC project, our target was C#, JS, and dot net. To do this on Windows requires wrapping each API in C++ code. Originally our APIs were manually written in CX/C++ (a Microsoft only C++ extension language), but maintaining the code base against a moving API set (i.e. WebRTC internal code) while not messing up the rules to convert to and from binded objects proved challenging.

The manual bindings were often subject to copy and paste errors and required heavily lifting every time a new API would be introduced or changed.

## Move to IDL for C++/WinRT

With the introduction of [C++/WinRT](https://github.com/Microsoft/cppwinrt), all the old wrapper CX/C++ code would need 
discarded entirely and replaced with a new implementation. Unlike CX/C++, C++/WinRT requires only modern standards compliant C++17 and uses no special language extensions. The C++ tooling provided from Microsoft takes [Microsoft IDL](https://docs.microsoft.com/en-us/windows/desktop/midl/midl-start-page) and generates a binary winmd file (metadata) describing the entire API set available (i.e. compiled IDL). Another tool takes the winmd file and generates modern templated managed C++ code and stubs needing to be implemented with glue code between the external APIs available and the internal implementation.

### Why not use Microsoft IDL?

The WebRTC UWP project does not use Microsoft IDL directly, but instead uses "SW" IDL input files. These IDL files are consumed by an IDL tool which generates Microsoft IDL as well as all the implementation needed for the C++/WinRT stub files and spits out new a new glue code that needs to be implemented between the exposed API language (in this case C++/WinRT) and the native API implementation.

This may seem redundant but there are key advantages.

### Advantages of "SW" IDL

SW IDL isn't restricted to being able to only generate and link with Microsoft based APIs. The current generator supports managed code like C++/WinRT and legacy CX/C++ generated wrappers. Other language bindings have been implemented too such as "C", platform C# dot net, and Python. More language support is under way.

### Why not SWIG/Corba/DCOM?

These IDL languages have served their era well but they are getting old now. SW modernizes both the IDL and their outputs supporting common modern and common language paradigms such as objects, interfaces, multiple inheritance, generics, events, promises and co-routines.

Other language facades are intentionally omitted, like language pointers, handles, low level constructs. This allows for the IDL to be both portable and modern without having to introduce risky and unstable API bindings.

SW IDL is very similar to what would be defined in WebIDL except it's not web specific. Nothing stops the SW IDL from generating SWIG IDL, corba, or other IDL based language bindings.

## Current implementation status

The current generated IDL output is dependent upon a library called "zsLib". This library handles basic async capabilities (events and queues) needed to make IDL work (even for languages that are single threaded). A new implementation is underway to replace this library with another library called SW library which will be an MIT licensed library with official corporate sponsorship (TBA). As part of this change, SW language binding support will expand and support both authoring and consuming in multiple languages as well as efforts for remote API calling.

## Dependency on SW

SW library a cross platform API set used for convenience in the generated C++ code for the bindings from C++ to other languages. This library is necessary to control asynchronous APIs and events and ensure every bound language is handled properly. As SW is a static library, non used routines will be linked out.

## Implementation Steps Overview

1. Build the SW IDL tool
2. Write IDL definitions
3. From IDL files+tool, generate authoring "stubs" ONCE only
4. Implement authoring stubs and check into git
5. Generate language bindings as needed for consumers of the authored APIs

## Example IDL files

https://github.com/webrtc-uwp/webrtc-apis/tree/Robin/m66-msidl-moved/idl  

## Example Glue implementation

https://github.com/webrtc-uwp/webrtc-apis/tree/Robin/m66-msidl-moved/windows/wrapper  
