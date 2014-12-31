ell
===

ℒ (ell) is a LISP core language.

ℒ can be used to implement Scheme and other LISP dialects. It specifies tail recursion and full continuations.

The semantics closely follow Scheme, but it adds maps, and a syntax [EllDN](https://github.com/boynton/elldn) that is
upwards compatible with JSON.

It defines a simple Virtual Machine (VM) that is fairly portable. C, Java, and Go implementations exist.

## Examples

The input and output in a REPL (Read-Eval-Print-Loop) session is shown. The "?" is the prompt, and if there is a result, then "=" precedes it.
When the result is *void* (as is the case with the print function), no result is printed.

### Hello world

	? (print "hello\n")
	hello

### Types and Literal Values

Data types all have a lexical format to specify literals:

	? type
	= #<primitive type>
	? (type "hello")
	= string
	? (string? "hello")
	= true
	? (type (type "hello"))
	= symbol
	? (type 'foo)
	= symbol
	? (type foo:) ;tailing colon is equivalent to quoting the symbol
	= symbol
	? (type true) ;true and false are bound to these symbols. Unlike Scheme's #t and #f
	= boolean
	? (type 23)
	= number
	? (type nil) ;nil is a symbol bound to a null value
	= null
	? (null? nil)
	= true
	? (type null) ;null is a synonym for nil
	= null
	? (type '()) ;empty list is also the same as nil
	= null
	? (type (cons 1 2)) ; a basic cons cell
	= pair
	? (type '(1 . 2)) ; all lists must be quoted to be taken literally. This is also a single cons
	= pair
	? (pair? '(1 . 2))
	= true
	? (list? '(1 . 2)) ; A list is either nil or a cons
	= true
	? (list? '())
	= true
	? (list? '(1 2))
	= true
	? (type [1 2 3])
	= vector
	? (equal? [1 2 3] [1, 2, 3]) ; commas are whitespace, this makes it JSON compatible
	= true
	? (type {x: 23 y: 57}) ; map with symbols as keys
	= map
	? (type {"foo" 23 "bar" 57}) ; map with string keys
	= map
	? (type {"foo": 23, "bar": 57}) ; JSON compatible for struct with string keys. Commas and colons are whitespace, keys are taken literally
	= map

### Defining and referencing top level bindings to data

	? (define h "hello") ; a string
	= "hello"
	? h
	= "hello"

### Defining a function

	? (define (add1 x) (+ 1 x))
	= #<function of 1 argument>
	? (add1 23)
	= 24
	? +
	= #<primitive +>

### Numbers
Numbers in Ell can be either integers or floating point numbers. Floats have the usual contagion.

### Strings

### Symbols

### Working with lists

	? (define p (cons 1 2))
	= (1 . 2)
	? (type p)
	= pair
	? (pair? p)
	= true
	? (car p)
	= 1
	? (cdr p)
	= 2
	? (define l (list 1 2))
	= (1 2)
	? (car l)
	= 1
	? (cdr l)
	= (2)
	? (cadr l)
	= 2
	? (cddr l)
	= nil
	? (length l)
	= 2
	? (length p)
	= -1
	? (null? (cddr l))
	= true
	? (append '(1 2) '(3 4))
	= (1 2 3 4)
	? (append '(1 2) '(3 . 4))
	= (1 2 3 . 4)
	? (append '(1 . 2) '(3 4))
	*** Error: argument 1 is not a proper list : 2
	? (reverse '(a b c))
	= (c b a)
	? (list-tail '(a b c) 2)
	= (c)
	? (list-tail '(a b c) 3)
	= ()
	? (list-tail '(a b c) 4) ;; scheme would produce an error here
	= ()
	? (list-ref '(a b c) 1)
	= b
	? (list-ref '(a b c) 3)
	*** Error: index out of range : 3

### Vectors
### Structs

### Identity and Equivalence
Ell has both identity and equivalence operators (`identity?` and `equal?`).

	? (identical? 'foo 'foo) ; identical makes sense for symbols, keywords, and singletons.
	= true
	? (equal? 'foo 'foo) ; equal is the more general comparison
	= true
	? (identical? foo: foo:)
	= true
	? (equal? foo: foo:) ;
	= true
	? (identical? nil nil)
	= true
	? (identical? "foo" "foo") ; just because strings look the same doesn't mean they are the same
	= false
	? (equal? "foo" "foo")
	= true
	? (identical? 2.5 2.5) ; floats are usually not identical, use equals? or =
	= false
	? (equal? 2.5 2.5)
	= true
	? (= 2.5 2.5) ; numeric equals behaves like equal?
	= true
	? (identical? [1 2 3] [1 2 3])
	= false
	? (equal? [1 2 3] [1 2 3])
	= true
	? (identical? {x: 23 y: 57} {x: 23 y: 57}
	= false
	? (equal? {x: 23 y: 57} {x: 23 y: 57}
	= true
	

	
## Types

### Data
Primitive data types include the following:

   * boolean - `true` or `false`
   * number - represented as a 64 bit value, either an integer or a floating point, i.e. `12` or `12.3`
   * string - a sequence of unicode values i.e. `"this is a string"`
   * symbol - a symbolic identifier, i.e. `foo`
   * cons - a 2-tuple of values, i.e. `(foo . bar)`
   * nil - the empty list, i.e. `nil` or `()`
   * list - either nil or a cons. This includes both proper and improper lists
   * vector - a compact linear sequence of values with constant time random access
   * map - a collection of keys mapping to values, i.e. `{foo: 23 "bar" 57}`

The external representation (data notation) is called [EllDN](https://github.com/boynton/elldn),
a superset of JSON.

For every type, there is a predicate function to test if a value is of that type.

### Other types

These types also have the corresponding predicates.

   * function - executable procedures that take arguments and return results. They can have side-effects, are not pure. Either primitive or bytecode
   * closure - a closure over a function and the current environment
   * port - an abstraction for reading/writing data
   * module - a separate compilation/definition unit that can be reused in other Ell programs.

## Execution

The environment is mutable, unlike Clojure. The basic data structures
are similar to "persistent" data structures, in that new instances are created and returned instead
of mutating in place. Struct and list comprehension functions are not lazy. Use Clojure if you want that.

Ell's execution is done with the Ell Virtual Machine ([LVM](https://github.com/boynton/ell/blob/master/lvm.md)).
It is a small vm, and presumes knowledge of things like Ell types and values. "Lisp Assembly Programs" (.lap files)
can be loaded and run directly.

In general, Ell programs are _read_ (they are data, after all), then _compiled_ to LVM instructions, and finally executed in the LVM. The LVM code can be represented externally as _lap_ code (Lisp Assembly Programs).

Compilation also supports simple macros, and performs macroexpansion before compiling to LVM code.

The execution is as follows:

   * Simple data types evaluate to themselves (nil, booleans, numbers strings)
   * Vectors elements get evaluated
   * Maps keys and values get evaluated
   * Symbols are used to look up values in the current lexical environment (variable references)
   * Lists are interpreted as function calls. The primitive expressions are:

      * `(lambda _args_ _expr_ _expr_ ...)` - returns a lambda expression closed over the current lexical environment
      * `(quote _datum_)` - the data value is taken literally
      * `(begin _expr_ _expr_ ...)` - the expressions are evaluated sequentially
      * `(if _cond_ _consequent_ _antecedent_)` - the _cond is evaluated, if true the the consequent is evaluated, else the antecedent
      * `(define _symbol_ _expr_)` - the expression is evaluated, and the result is assigned to the symbol in the current environment's scope
      * `(set! _symbol_ _expr_)` - the expression is evaluated, and the result is assigned to the already-defined symbol in the current environment
   * The set of valid expressions can be extended with macros, which expand into primitive forms
   * All other Lists are interpreted as function calls. The first element is the function, applied to the remaining elements (arguments). The resulting value is the result of the function call.	Functions may be part of the basic environment (primitives - usually defined in the underlying implementation language), or the result of a lambda expression. Also, vectors, maps, and keywords are executable:

      * `(_vector_ _index_)` - looks up an element of a vector with the specified index
      * `(_map_ _key_)` - looks up a value in a struct for the given key.
      * `(_keyword_ _map_)` - looks up the value in a map for the given keyword, i.e. `

## Core functions

   * `(quasiquote _expr_)` - the expression is quoted, but allowing unquote and unquote-splicing to occur within. Typically bound to the "`" reader macro.
   * `(unquote _expr)` - unquotes the expression within a quasiquote
   * `(unquote-splicing _expr)` - unquotes the expression within a quasiquote and splices it as a list into the current quasiquoted expression
   * `(macro args . body)` - creates a macro, which can be bound with define, i.e. (define defn (macro (name args . body) `(define ,name ,args ,@body)))
   * `(cons _expr1_ _expr2)` - creates a cons cell made of the 2 values
   * `(car _pair_)` - returns the first element of a pair
   * `(cdr _pair_)` - returns the second element of a pair
   * `(make-vector _list_)` - returns a vector with the elements determined by the list
   * `(make-struct _list_)` - returns a map made from the list. The list must have an even number of elements, keys and values are taken alternatively
   * `(sequence? _expr_)` - returns true if the value implements the Sequence interface, i.e. is a list, vector, or struct
   * `(length _sequence_)` - this counts the sequence (might be expensive!)
   * `(first _sequence_)` - returns the first object in the sequence
   * `(rest _sequence_)` - returns a sequence representing the rest of the sequence, or nil
   * `(seq obj)` returns a sequence for the object, or nil if none can be created.


