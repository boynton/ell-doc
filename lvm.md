LVM
===

The ℒ language can be implemented in terms of the small virtual machine described here.

The fundamental VM data structure is a _frame_. Frames are linked to lexically scoped parent frames.
Each frame contains a vector of bindings of local variables to values.
This linked list of frames is maintained during execution, representing the call stack,
and accessing a local variable is done by
first indexing into the environment to find the right frame, and then indexing into that frame's
bindings to access the value.
When a closure is formed, the lexical frame is preserved, and when that closure is executed, that
frame is pushed onto the call stack.

Additionally, a frame holds the a snapshot of the state of execution of the calling function (the previous
program counter (PC), the previous environment, and the previous code object), and a link to that caller's
frame.

The VM consists of the following registers to keep track of execution:

* constants - a vector of known constants for the currently executing function, indexed into by integer
* stack - an array of values used as the value stack
* sp - the current index into the top of the stack
* ops - the array of bytecodes being executed
* pc - the current index into the ops, where current execution is at
* environment - the current frame of execution

===

* LITERAL _lit_ - push the value in the constant pool at offset _lit_ onto the stack
* POP - pop and discard the top value on the stack
* JUMPFALSE _offset_ - test the value at the top of the stack and branch the specified offset if the value is false.
* JUMP _offset_ - add the offset to the _pc_ and continue execution
* CALL _argc_ - pop the top value off the stack (a function) and call it, passing the specified number of args	
* TAILCALL _argc_ - pop the top value off the stack (a function) and call it, passing the specified number of args. The stack frame is reused.
* RETURN - return from a CALL or TAILCALL
* CLOSURE _lit_ - create a closure on the specified function over the current environment. The constant pool is indexed into by _lit_ to access the function.
* CALLCC - captures the current continuation of execution, and calls function on the top of the stack with it as a single argument.
* LOCAL _i_ _j_ - using the _i_th frame in the environment, push the _j_th value onto the stack.
* GLOBAL _i_ - push the value contained in the _i_th symbol in the constant pool to the top value on the stack.
* SETLOCAL _i_ _j_ - set the value of the _j_th binding in the _i_th frame of the environment to the value on the top of the stack.
* SETGLOBAL _i_ - set the global defined by the _i_th constant to have the value on the top of the stack. An error is the global is not yet set.

globals themselves should be defined in namespace. This makes it a non-instruction, but rather an attribute on a code object. Or: the final frame?
	!! would be nice to simply treat globals as the outermost frame. Could extract from the env.
	!! a REPL would want to define things in that top level environment, including compiling code. Hmm. Maybe
	-> big performance boost to have that top frame accessible. It is like "extern", needed for "free variables". Like system functions.
	The symbol table is essentially how to link modules. Clojure adds a local namespace for this, with no predefined global namespace. I.e. you
		must import system symbols into your space. This seems pretty reasonable, actually
	
* DEFGLOBAL _i_ - define the global defined by the _i_th constant to have the value on the top of the stack.
	? don't really need this. Have setglobal be the primitive, without the check if exists or not. Define (define) and (set!) to generate the right code
		including the check if needed.

* PRIMITIVE _i_ _argc_ - the _i_th constant (a native callable object, i.e. a Method or Function) is called. Argc indicates how many arguments to take off the stack. This opcode is an optional optimization that statically binds global functions to opcodes at compile time.



