# ABI notes, Jim Kupsch, 2021-09-15

## Terminology

- ***Using ABI*** - dependent on system or kernel ABI
- ***imported ABI contamination*** - place where a library's ABI is incorporated in another library or executable

## Notes

1. programming language
    1. names
        1. allowed names & character encoding determined by language
        1. scope determines accessibility of name
            1. global
            1. class
            1. function/method
            1. block
        1. may be nested
            1. classes
            1. namespaces
        1. visibility 
            1. within compilation unit only
            1. visible to other compilation units via declarations
    1. types
        1. can be named
        1. data
            1. fundamental types
                1. integers
                    1. encoding
                    1. signed/unsigned/other
                    1. size
                1. floating point
                    1. encoding
                    1. size
            1. derived types
                1. processor specific vector types
                1. arrays
                1. class/struct/record
                    1. collection of named data objects & methods
                    1. derived class
                        1. extends another class with data and methods
                    1. polymorphic (virtual) class
                        1. derived class
                        1. virtual methods determined at runtime
                1. union
                    1. collection of named data objects & methods
                    1. each shares the same the same memory
                1. pointers
                1. enums
                    1. stored as some int type
                    1. has enumerates that map name to constant
                1. bit fields
                1. complex number
            1. type qualifiers
                1. const
                1. volatile
                1. atomic
                1. thread local
        1. function/method
            1. return type
            1. optional type qualifiers
            1. methods have an implicit parameter pointing to the object
            1. parameters
                1. typed parameters
                1. optional variadic parameters
                    1. type and number may be fixed (e.g. open)
                    1. type and number determined at runtime (e.g. printf)
    1. language runtime support library
        1. dynamic memory management
            1. malloc, std::allocator
        1. runtime type information
        1. exceptions
        1. atexit functionality
    1. object
        1. has a type and size
        1. located in memory or register(s)
        1. represents a
            1. named variable
            1. temporary object
            1. parameter
            1. dynamically allocated
    1. function
        1. has a type
        1. statements define the function's data transformations and calls to other functions
    1. declaration
        1. maps a name to a type
    1. definition
        1. also a declaration
        1. of a type
        1. of a variable
            1. is a named object
        1. of a function
    1. macros
        1. has a name
        1. has substitution text
        1. optionally parameterized
        1. name in source code is textually replace by substitution text before compilation

1. source code
    1. written in a programming language
    1. textual description of a program
    1. names defined
        1. types
        1. functions
        1. variables
        1. macros
    1. names declared
        1. types
        1. functions
        1. variables
        1. macros
        
1. architecture/processor
    1. the processor that the software will run on
    1. has an instruction set architecture (ISA) that is the encoding and semantics of instructions
    1. may have levels where new instructions are added or removed
    1. directs mapping of types and properties
        1. based on efficient hardware support for size and encoding
        1. alignment requirements based on hardware requirements and efficiency
        1. use of emulation software if not available

1. API
    1. software interface that allows a program to use libraries
    1. declarations present in a library for use by external code
        1. functions
        1. types
        1. data objects
    1. public API is a stable interface to a libraries functions and data allowing the implementation to be enhanced and bugs fixed
    1. no modification required to user code if
        1. types in API are changed to compatible types
        1. additional functions and objects can be added
    1. semantics of functions and data described in documentation

1. ABI
    1. conventions that allow for the interoperability and compatibility of binary code objects to form an executable that can be run on an OS and architecture.
        1. allows interoperability between
            1. compiler vendors and versions
            1. kernel versions
            1. library implementations and versions
    1. categories
        1. kernel/OS ABI
            1. interface to make system calls to perform OS system calls from a user space process
            1. typically different from the function call ABI in the system ABI
            1. user code will rarely directly use this ABI, instead accessing system calls via a library such as libc
            1. for each OS syscall
                1. syscall number
                1. semantics of functionality
                1. number of parameters & return value
                    1. each has a data type
                    1. may be input or output
                    1. parameter and return type semantics
            1. syscall ABI calling convention
                1. how privileged call to OS is made
                1. how syscall number is passed
                1. how parameters are passed
                1. how results are returned
                1. modification to process state the syscall is allowed to make
                    1. stack
                    1. registers
                    1. flags
        1. system ABI
            1. conventions to call functions, use libraries, begin/end execution
            1. library/executable format
            1. mapping of programming language types to architecture types
            1. function call interface
                1. how parameters are passed
                    1. registers
                    1. stack (memory)
                1. how results are returned
                1. requirements of the caller
                    1. stack alignment
                    1. flags/mode requirements
                1. requirements of the callee
                    1. flag/mode requirements
                    1. register/memory modifications
                1. modification allowed by the callee to CPU state & stack
                1. mechanism to transferred control to function
                    1. stack usage
                    1. find address of functions entry point
                        1. direct call if in same object and not exported
                        1. if to external object or exported
                            1. GOT, PLT
            1. register usage
            1. loading
                1. how kernel loads executable into memory to begin execution
                1. initial state setup
                1. optionally loads loader
                1. transfers control to object or loader to complete initialization
            1. initial process state
                1. CPU flags
                1. registers
                    1. meanings
                1. stack
                    1. meaning of data
                1. data from user space and OS
                    1. argv array
                    1. environment variables
                    1. auxiliary vector
            1. rules for signals handlers
            1. mapping of hardware exceptions to signals
            1. threading
                1. thread local data
            1. exception handling
                1. throwing, catching
                1. stack unwinding
            1. CPU, stack, memory conventions
                1. special registers
                    1. GOT
                    1. frame/base pointer
                    1. linkage
                1. stack
                    1. red zone
                    1. signal
        1. library ABI
            1. created by linking together object files into a library that were created by compiling source code
            1. manifestation of library API as realized in the object code using the system ABI
                1. exported typed functions
                1. exported typed data

1. compilation
    1. transforms source file(s) to object file
    1. generally source files are compiled separately (a compilation unit)
    1. generates code
        1. use target processor's ISA
        1. ***using ABI*** conventions for external functions calls/definitions
            1. ***imported ABI contamination*** of symbol use if from another library
        1. accessing data ***using ABI*** conventions for the type
            1. ***imported ABI contamination*** if type is from another library
        1. functios internal to this object file can be optimized by not following system ABI
        1. for imported-inlined functions
            1. ***imported ABI contamination*** if from another library
        1. non-static local variable
            1. generated by code for the function at runtime
            1. ***imported ABI contamination*** if type from another library
        1. inlined runtime support library functions
            1. ***imported ABI contamination*** from language runtime
    1. generates data
        1. ***using ABI*** conventions for
            1. types
            1. size
            1. layout
            1. alignment
        1. ***imported ABI contamination*** if definition from another library
        1. global variable data
            1. visibility:  outside object
            1. per executable or per thread
        1. static function data
            1. visibility:  local to this object
            1. per executable or per thread
    1. maps source code names to object file symbols ***using ABI***
    1. maps source code types to code ***using ABI***
        1. ***imported ABI contamination*** if from another library
    1. linkage information
        1. external symbols referenced
        1. how to patch object to allow symbol to be used
    1. synthesizes function/code needed at runtime
        1. implicitly generated functions such constructors/destructors
        1. template instantiation
        1. exception handlers
        1. initialization/finalization
        1. multiple versions of functions or blocks
            1. parallelization
            1. architecture feature enabled
            1. resolver functions to select "best" implementation at runtime
        1. partial inlining
    1. synthesizes runtime necessary data structures
        1. RTTI
        1. exceptions
    1. debug data if needed
    1. data to support link time optimization if needed
    1. macros
        1. textual replacement of a word in the source file
            1. happens before processing by compiler
            1. possibly parameterized
            1. value can be conditionally determined at compile time
        1. names are not present in object file, just artifacts of expansion
        1. ***imported ABI contamination*** if macor from another library is exanded

1. object file
    1. compiled source code
    1. linking information
        1. symbols defined
            1. have course type: function or object (data)
            1. have scope: local or global (exported)
            1. may be versioned
        1. symbols required from other objects/libraries
            1. where in memory to patch is symbols address
    1. no need for types other than symbol names
    1. debug information
    1. link time optimization (LTO) data

1. linker
    1. creates executable or library from object files and external libraries
    1. relocates object references to function and data
    1. combines from each object file
        1. code
        1. data
        1. debug information
        1. LTO data
        1. external symbols
    1. determines list of symbols to export
    1. symbols can be versioned based on object file symbols and map file
    1. external versioned symbols based on external library passed to linker
    
1. executable
    1. file that the OS kernel can load into memory to form a process
    1. may also be a library (rare)
    1. contains
        1. required symbols
        1. external libraries using SONAME to load to resolve needed symbols

1. library
    1. dynamic library or executable
        1. single object
        1. has an SONAME (library name and ABI version)
    1. static library
        1. collection of object files
    1. contains
        1. list of exported symbols
        1. required symbols
        1. external libraries using SONAME to load to resolve needed symbols

1. loader
    1. used if dynamic linking is required
    1. loads required libraries recursively based on needed SONAMEs
    1. initializes dynamic linking data structures
    1. patches data using resolved symbols and linkage data
    1. computes runtime determined symbol values
    1. transfers control to executable's entry point

1. problems
    1. if ABI doesn't cover feature of programming language or compiler
    1. if ABI doesn't cover architecture feature
    1. if tool chain does not follow system or kernel ABI correctly

1. issues
    1. information to validate library ABI is neither in the libraries or executable as it is discarded by the compiler and linker
    1. needed symbols do not include the SONAME of the library that they should be provided by; symbols may be inadvertently provided by the incorrect library
