ell
===

ℒ (ell) is a Lisp core language. It provides a basic List data structure, and also adds
Maps, and Arrays. Primitive types include Strings, Symbols, Numbers, and Booleans.

ℒ can be used to implement Scheme and other lisp dialects. It supports namespaces, so that such
languages can be safely built, and specifies tail recursion and full continuations.

The environment is mutable (Symbols have global values), unlike Clojure. The basic data structures
are similar to "persistent" data structures, in that new instances are created and returned instead
of mutating in place. Map and list comprehension functions are not lazy. Use Clojure if you want that.

Although it is largely defined in terms of its small 14 instruction virtual machine, the actual core
language definition is in terms of S-exprs and lisp syntax, not the instructions of the VM. Alternative
implementations might add instructions (to both VM and Compiler), or compile to another target altogether.

Core forms include the following:

   * nil - no value is taken literally
   * true, false - the boolean values true or false are taken literally (the same as scheme's #t and #f
   * 123, 123.456 - numbers are taken literally. Floats and integers (both with 64 bit precision) are supported.
   * "my string" - the string is taken literally
	* :foo, foo: - a leading or trailing colon means the symbol is in the special keyword namespace. Keywords evaluate to themselves.
   * _symbol_ - For any other symbol, the value of the symbol in the current environment is used
   * (1 2 3) - parentheses create lists, which are linked cons cells. Lists must be quoted, or they are taken as a function call form
   * [1 2 3] - brackets create vectors, always used literally
	* {one: 1 two: 2 "three" 3} - braces create maps, taken literally. The keys are arbitrary, but keywords are common 
   * (lambda _args_ _expr_ _expr_ ...) - returns a lambda expression closed over the current lexical environment
   * (quote _datum_) - the data value is taken literally
   * (begin _expr_ _expr_ ...) - the expressions are evaluated sequentially
   * (if _cond_ _consequent_ _antecedent_) - the _cond is evaluated, if true the the consequent is evaluated, else the antecedent
   * (define _symbol_ _expr_) - the expression is evaluated, and the result is assigned to the symbol in the current environment's scope
   * (set! _symbol_ _expr_) - the expression is evaluated, and the result is assigned to the already-defined symbol in the current environment

Core functions allow languages to be built:

   * (macro args . body) - creates a macro, which can be bound with define, i.e. (define defn (macro (name args . body) `(define ,name ,args ,@body)))
   * (cons _expr1_ _expr2) - creates a cons cell made of the 2 values
   * (car _pair_) - returns the first element of a pair
   * (cdr _pair_) - returns the second element of a pair
   * (make-vector _list_) - returns a vector with the elements determined by the list
   * (make-set _list_) - returns a set of the unique given elements. i.e. (make-set [1 2 3])
   * (make-map _list_) - returns a map made from the list. The list must have an even number of elements, keys and values are taken alternatively
   * (sequence? _expr_) - returns true if the value implements the Sequence interface, i.e. is a list, vector, set, or map
   * (length _sequence_) - this counts the sequence (might be expensive!)
   * (first _sequence_)
   * (rest _sequence_)
	
   * (quasiquote _expr_) - the expression is quoted, but allowing unquote and unquote-splicing to occur within
   * (unquote _expr) - unquotes the expression within a quasiquote
   * (unquote-splicing _expr) - unquotes the expression within a quasiquote and splices it as a list into the current quasiquoted expression

