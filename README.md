ell
===

ℒ (ell) is a LISP core language, with semantics similar to Scheme, but adds Keywords, Structs, and user-defined Types.

It defines a simple Virtual Machine (VM) that is has C, Java, and Go implementations, with some plans to target
other back ends.

## Types

All types are represented as symbolic identifiers withi surrounding angle brackets, i.e. <foo>.

### Data
Primitive data types, which have syntax for input/input, include the following:

   * boolean - `true` or `false`
   * character - a single unicode character `#\A` or `#\space`
   * number - represented as a 64 bit float i.e. `5` or `10.3`
   * string - a sequence of characters i.e. `"this is a string"`
   * symbol - a symbolic identifier, i.e. `foo`
   * keyword - a symbolic identifier, self-evaluating, used as keys and as functions, i.e. `foo:`
   * type - a symbolic identifier representing metatype, i.e. `<foo>`
   * list - a 2-tuple of a value and another list (no dotted pairs), i.e. `(1 2 3)`
   * array - a compact linear sequence of values with constant time random access, i.e. `[1 2 3]`
   * struct - a collection of keys mapping to values, i.e. `{foo: 23 "bar" 57}`
   * instance - a user-typed value on any other value, i.e. `#<point>{x: 23 y: 57}`
   * null - no value, i.e. `null`

The external representation (data notation) is called [EllDN](https://github.com/boynton/elldn),
a superset of JSON.

For every type, there is a predicate function to test if a value is of that type. All types are disjoint.

New data types can be created with the `deftype` and `defstruct` macros, which set up a constructor and predicate
to define new instances of the defined type.

The most general form is deftype, in which is like defining a predicate for a type. Other supporting functions are
then generated:

   ? (type "hello")
   = <string>
   ? (string? "hello")
   = true
   ? (deftype uuid (s) (and (string? s) (= (string-length s) 36)))
   = <uuid>
   ? (uuid)
    *** Wrong number of args to <function uuid> (expected 1, got 0)
   ? (def u (uuid "0f1a4749-5e58-11e5-a68f-003ee1be85f9")
   = #<uuid>"0f1a4749-5e58-11e5-a68f-003ee1be85f9"
   ? (type u)
   = <uuid>
   ? (uuid? u)
   true
   ? (string? u)
   false
   ? (type (value u))
   <string>

For structured types, defstruct is provided, with a more convenient constructor:

   ? (defstruct rect x: 0 y: 0 w: 0 h: 0)
   = <rect>
   ? (rect)
   = #<rect>{x: 0 y: 0 w: 0, h: 0}
   ? (def r (rect w: 20 h: 10))
   = #<rect>{x: 0, y: 0 w: 20 h: 10}
   ? (type r)
   = <rect>
   ? (rect? r)
   true
   ? (as-rect {w: 20 h: 10})
   *** cannot convert to <rect>, missing field: x


### Other types

These types also have the corresponding predicates.

   * function - executable procedures that take arguments and return results. They can have side-effects, are not pure. Either primitive, or a closure over compiled bytecode
   * input, output - an abstraction for reading/writing data

## Function forms

In addition to traditional lambda definitions, with explicit arguments and/or "rest" arguments, optional named arguments with defaults, and keyword arguments, are also supported:

	? (defn f (x y) (list x y))
	= <function of 2 arguments>
	? (f 1 2)
	= (1 2)
	? (defn f (x & rest) (list x rest))
	= <function of 1 or more arguments>
	? (f 1 2)
	? (defn f args args)
	= <function of 0 or more arguments>
	? (f 1 2 3)
	= (1 2 3)
	? (deffn f (x [y]) (list x y))
	= <function of 1 or more arguments>
	? (f 1 2)
	= (1 2)
	? (f 1)
	= (1 null)
	? (defn f (x [(y 23)]) (list x y))
	= <function of 1 or more arguments>
	? (f 1)
	= (1 23)
	? (f 1 2)
	= (1 2)
	? (defn f (x {y: 23 z: 57}) (list x y z))
	= <function of 1 or more arguments>
	? (f 1)
	= (1 23 57)
	? (f 1 2)                                                                                               
	 *** Bad keyword arguments: [2] 
	? (f 1 y: 2)
	= (1 2 57)
	? (f 1 z: 2)
	= (1 23 2)
	? (f 1 z: 2 y: 3)
	= (1 3 2)

## Generic functions and methods
Multimethod dispatch is also supported, based on the types of arguments.

   ? (defgeneric area (shape))
   = <function area>
   ? (area r)
    *** Generic function area has no matching method for: (<rect>)
   ? (defmethod area ((r <rect>)) (* (w: r) (h: r)))
   = area
   ? (area r)
   = 200
   ? (defstruct circle x: 0 y: 0 r: 0) 
   = <circle>
   ? (def c (circle r: 10))
   = #<circle>{x: 0, y: 0, r: 10}
   ? (area c)
    *** Generic function area has no matching method for: (<circle>)
   ? (def pi 3.141592653589793)
   = 3.141592653589793
   ? (defmethod area ((c <circle>)) (* pi (r: c) (r: c)))
   = area
   ? (area c)
   = 314.1592653589793

Multiple arguments can be specialized:

   ? (defgeneric encounter (x y))
   ? (defstruct bunny)
   ? (defstruct lion)
   ? (defmethod encounter ((b <bunny>) (l <lion>)) 'run)
   ? (defmethod encounter ((l <lion>) (b <bunny>)) 'eat)
   ? (defmethod encounter ((l1 <lion>) (l2 <lion>)) 'fight)
   ? (defmethod encounter ((b1 <bunny>) (b2 <bunny>)) 'mate)
   ? (def b1 (bunny))
   ? (def b2 (bunny))
   ? (def l1 (lion))
   ? (def l2 (lion))
   ? (encounter b1 b2)
   = mate
   ? (encounter b1 l1)
   = run
   ? (encounter l1 b1)
   = eat
   ? (encounter l1 l2)
   = fight

## Execution

The environment is mutable, unlike Clojure. However, the basic data structures are used
in a way similar to "persistent" data structures, in that new instances are created and returned instead
of mutating in place. Struct and list comprehension functions are not lazy. Use Clojure if you want that.

Ell's execution is done with the Ell Virtual Machine ([LVM](https://github.com/boynton/ell/blob/master/lvm.md)).
It is a small vm, and presumes knowledge of things like Ell types and values.

In general, Ell programs are _read_ (they are data, after all), _expanded_ based on  currently defined macros, then
_compiled_ to LVM instructions, and finally executed in the LVM. The LVM code can be represented externally, also
as valid a Ell data structure, and be loaded directly (read and execute).

The execution is as follows:

   * Simple data types evaluate to themselves (null, boolean, char, number, string)
   * keywords and types evaluate to themselves
   * Array elements get evaluated
   * Struct keys and values get evaluated
   * Symbols are used to look up values in the current lexical environment (variable references)
   * Lists are interpreted as function calls. The primitive expressions are:

      * `(lambda` _args_ _expr_ _expr_ ...`)` - returns a lambda expression closed over the current lexical environment
      * `(quote` _datum_`)` - the data value is taken literally
      * `(begin` _expr_ _expr_ ...`)` - the expressions are evaluated sequentially
      * `(if` _cond_ _consequent_ _antecedent_`)` - the _cond is evaluated, if true the the consequent is evaluated, else the antecedent
      * `(define` _symbol_ _expr_`)` - the expression is evaluated, and the result is assigned to the symbol in the current environment's scope
      * `(set!` _symbol_ _expr_`)` - the expression is evaluated, and the result is assigned to the already-defined symbol in the current environment
      * `(callcc` _fun_`)` - capture the current continuation and call the function with it as a n argument

   * The set of valid expressions can be extended with macros, which expand into primitive forms
   * All other Lists are interpreted as function calls. The first element is the function, applied to the remaining elements (arguments). The resulting value is the result of the function call.	Functions may be part of the basic environment (primitives - usually defined in the underlying implementation language), or the result of a lambda expression.

## Core functions

   * `(quasiquote` _expr_`)` - the expression is quoted, but allowing unquote and unquote-splicing to occur within. Typically bound to the "`" reader macro.
   * `(unquote` _expr_`)` - unquotes the expression within a quasiquote
   * `(unquote-splicing` _expr_`)` - unquotes the expression within a quasiquote and splices it as a list into the current quasiquoted expression
   * `(macro` args & body`)` - creates a macro, which can be bound with define, i.e. (define defn (macro (name args & body) `(define ~name ~args ~@body)))
   * `(cons` _expr_ _list_`)` - creates a cons cell made of the 2 values. Note that the cdr must be a list
   * `(car` _list_`)` - returns the first element of a list
   * `(cdr` _list_`)` - returns the rest of the list
   * `(list?` _obj_`)` - returns true if the object is a list
   * `(array` _e1_ _e2_ ...`)` - returns an array with the specified elements
   * `(struct` _k:_ _v_ ...`)` - returns a struct with the specified key//value pairs. The list must have an even number of elements, keys and values are taken alternatively. Keys are usually keywords, but can be any object

## Examples

The input and output in a REPL (Read-Eval-Print-Loop) session is shown. The "?" is the prompt, and if there is a result, then "=" precedes it.

### Hello world

	? (print "hello\n")
	hello

### Types and Literal Values

Data types all have a lexical format to specify literals:

	? type
	= #<function type>
	? (type "hello")
	= <string>
	? (string? "hello")
	= true
	? (type (type "hello"))
	= <type>
	? (type 'foo)
	= <symbol>
	? (type foo:) ;tailing colon indicates a keyword instead of a symbol
	= <keyword>
	? (type <foo>) ; angle brackets indicate a type instead of a symbol
	= <type>
	? (type true) ;true and false are bound to these symbols. Unlike Scheme's #t and #f
	= <boolean>
	? (type 23)
	= <number>
	? (type null) ;null is a symbol bound to a null value
	= <null>
	? (type '()) ;empty list is not the same as null
	= <list>
	? (type (cons 1 '())) ; a basic cons cell
	= <list>
	? (type '(1 2)) ; all lists must be quoted to be taken literally.
	= <list>
	? (list? '(1 2))
	= true
   ? (empty? '(1 2))
   = false
   ? (empty? '())
   = true
	? (list? '())
	= true
	? (list? null)
	= false
	? (list? '(1 2))
	= true
	? (type [1 2 3])
	= <array>
	? (equal? [1 2 3] [1, 2, 3]) ; commas are whitespace, this makes it JSON compatible
	= true
	? (type {x: 23 y: 57}) ; map with symbols as keys
	= <struct>
	? (type {"foo" 23 "bar" 57}) ; map with string keys
	= <struct>
	? (type {"foo": 23, "bar": 57}) ; JSON compatible for struct with string keys. Commas and colons are whitespace, keys are taken literally
	= <struct>

### Defining and referencing top level bindings to data

	? (def h "hello") ; a string
	= "hello"
	? h
	= "hello"

### Defining a function

	? (defn add1 (x) (+ 1 x))
	= <function add1>
	? (add1 23)
	= 24
	? +
	= <function +>

### Defining a macro

   ? (defmacro foo (x) `(string "a quoted list around the value " ~x " which was passed in"))
   = <function foo>
   ? (macroexpand '(foo 57))
   = (string "a quoted list around the value " 57 " which was passed in")
   ? (foo 57)
   = "a quoted list around the value 57 which was passed in"


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



