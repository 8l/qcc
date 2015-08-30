A Quick C Compiler
http://c9x.me/qcc/

What is this ?

This is a tiny C compiler, I wrote it in a very restricted dialect of OCaml. It is in the same family as otcc but will work for the x86_64 architecture. My medium term goal is to port it into the C dialect it compiles, so I can bootstrap the thing. It will generate dynamically linked ELF files for the 64bits architecture x86_64 (AMD64 or IA-32e). It has no intermediate language and generates code as it parses the C source file. This makes it very fast, unsafe and unoptimizing. Code generated sucks, no doubt, but the compiler is very short and understandable.

Unless your code is dead wrong, QCC should remain very silent and produce an output file. In its current state, it will happily compile the expression 1 = 2, depending on the context, the semantics will differ.

I wrote it to learn x86 64bits machine code and ELF files with dynamic linking. It is very easily extensible and hackable, you are welcome to add your own set of improvements.

    It is alive: download qcc.ml (~20K).
    Or see it in color qcc.ml.html.
    See also a small useful example which qcc can compile qccx.c.
    The NOTES file contains some gathered information about ELF and machine code. 

Supported features
The C subset supported features the following language constructs.

    Local and global functions with clean scoping.
    The statements if, while, for, break and return.
    Nested blocks with proper declaration support.
    Forward references (recursion).
    String literals.
    C unary and binary operators with proper precedence and associativity.
    Shortcutting && and ||.
    Post increment and decrement.
    Type int (caution, it is 64 bits wide).
    Pointer types (through explicit casts).
    Function pointers.
    External function calls and variables (through dynamic linking). 

Inner machinery

Disgusting.

I used the architecture one can find in otcc, i.e. the dumbest possible. Just parse while emiting code. You can find a small and quite readable example of this technique in TempleOS' mini compiler; reading it will help you understand how qcc works.

One neat trick I pulled from otcc is the symbol patching mechanism. Namely, when we don't know the value of a given symbol yet while emitting code we use the bits dedicated to this value in the machine code to store a linked list of all locations to be patched with the given symbol value. See the code of the patch function for details about that.

What makes compiling for 64bits architecture harder is that integer are 32bits while pointers are 64bits. That is really bad if we want to compile properly because this implies the need to keep track of types. Fortunately, we don't want to compile properly so I made integers 64bits wide. This does not cause much trouble with the ABI.

The layout of produced ELF files includes a single code segment. It will contain global variables and string literals at the beginning and code just after. Yes, data is in the code section. Because we don't know how many global variables the program will use we patch memory accesses with the right addresses only when we read the whole input file. Unresolved symbols will be handled by the dynamic linker.

The handling of lvalues is a bit cleaner than in otcc since I wanted the parsing to be smooth. This allows proper post-increment and pre-increment treatment. My solution makes use of a fun system of late patching of the generated code, see patchlval for details. A friend of mine called this speculative compilation. 
