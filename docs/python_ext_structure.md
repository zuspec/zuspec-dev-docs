
# Python extension structure

## File structure
There are two sets of files in an extension:
- Cython extension files (python/core.pyx, python/<pkg>/core.pxd, python/<pkg>/decl.pxd)
- Helper C++ files (python/*.cpp) that implement APIs that call Python from C++

### Cython extension files
The three Cython extension files has specific roles:
- The python/<package>/decl.pxd file specifies class types and method prototypes matching 
  classes and methods in the C++ API's pure-virtual API 
- The python/<package>/core.pxd file defines Cython classes that implement a wrapper around the 
  C++ class pure-virtual APIs. This declarations are kept separate of the implementation
  in the .pyx file such that other Cython extensions can use the classes as the Cython/C++
  level.
- The python/core.pyx file provides the implementation for the wrapper classes defined in core.pxd.
  This file is compiled to a native extension that knows how to use the C++ API

## Factory
Each extension has a Factory that is responsible for initializing the C++ library that
the extension wraps. 

## Context
Many C++ libraries have a 'IContext' class that maintains state and constructs objects
that will belong to that context. The Cython extensions implement a Context class
and implement factory methods for objects managed by the context. Think of Context
as a factory for library objects.

## Handling Enum Types
- decl.pxd defines a cdef enum for each C++ enum type
- Each enumerator is listed, along with the fully-qualified enumerator name

When passing a value of enumerated type from Python to the C++ API:
- Convert the Python enum value to an cdef int
- Use a static cast to convert the int to the enum type

## Visitor
C++ libraries frequently have a visitor interface that can be implemented by clients. 
The Python extension exposes this Visitor interface as a Python class from which clients
can extend.
- The VisitorProxy class is a C++ class that inherits from the C++ library's VisitorBase.
  When a visit* method in this class is invoked, it forwards the call to a public 
  C method implemented by the Cython extension (VisitorProxy_visit*). The public C 
  method is passed a handle to the Python visitor object and a handle to the C++ object.
- VisitorProxy_visit* method: This method receives a handle to the visitor object
  and the C++ object. It constructs a Cython wrapper object for the C++ object, and
  passes that to the relevant visit* method on the Python visitor class.


## Wrapper Builder
The C++ API will often return 'base' C++ handles. The Cython extension must create
a Python object corresponding to the least-base handle. The WrapperBuilder class
performs this task by implementing the Visitor API.

When the WrapperBuilder's 'mkObj' method is called:
- It invokes the C++ object's "accept" method such that the object calls its
  visit method
- The object's visit method saves the Python wrapper class that the visitor
  passes to it.

## Memory Management
Memory management is critical with this extension strategy. The Python wrapper classes track
whether they 'own' the C++ object using a flag (_owned). If the Python wrapper "owns"
the C++ object when the wrapper is disposed, then it will delete the C++ object as well.

When a Python wrapper is passed to a method that takes on ownership of the object,
the method must:
- Check that the object being passed 'owns' the C++ object and raise an error if not
- Set the 'owned' flag False so that the wrapper does not delete the C++ object

