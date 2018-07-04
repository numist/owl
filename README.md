Bluebird is an experimental parser generator.  It targets the class of *visibly pushdown languages*, which sit between regular languages and context-free languages on the formal language hierarchy.

Unlike traditional context-free-language parser generators, bluebird can tell you about ambiguities directly in terms of the language it recognizes:

-this is where the colorful image goes-

Here's an example of a bluebird grammar for a language with arithmetic and variable assignment:

```
program = statement*
statement =
  identifier '=' expression : assignment
  'print' expression : print
expression =
  identifier : variable
  number : number
  [ '(' expression ')' ] : parens
 .operators prefix
  '-' : negate
 .operators infix left
  '*' : multiply
  '/' : divide
 .operators infix left
  '+' : add
  '-' : subtract
```

Check out more examples in the [example/](example/) directory.

## motivation

…

## how to

You can build the `bluebird` tool from this repository using `make`:

```
$ git clone https://github.com/ianh/bluebird.git
$ cd bluebird/
$ make
```

Run `make install` to copy the `bluebird` tool into `/usr/local/bin/bluebird`.

Bluebird has two modes of operation&mdash;**test mode** and **compilation mode**.

In **interpreter mode**, bluebird reads your grammar file, then parses standard input on the fly, producing a visual representation of the parse tree as soon as you hit `^D`:

```
$ bluebird grammar.bb
...
```

You can specify a file to use as input on the command line with `--input` or `-i`:

```
$ bluebird grammar.bb -i file
...
```

In **compilation mode**, bluebird reads your grammar file, but doesn't parse any input right away.  Instead, it generates C code with functions that let you parse the input later:

```
$ bluebird -c grammar.bb -o parser.h
```

You can `#include` the generated parser into a C program:

```
...
```


## rules and grammars

Rules in bluebird are written like regular expressions with a few extra features.  Here's a rule that matches a comma-separated list of numbers:

```
number-list = number (',' number)*
```

Note that bluebird operates on tokens (like `','` and `number`), not individual characters.

To create a parse tree, you can write rules that refer to each other:

```
variable = 'var' identifier (':' type)?
type = 'int' | 'string'
```

Rules can only refer to later rules, not earlier ones: plain recursion isn't allowed.  Only two restricted kinds of recursion are available: *guarded recursion* and *expression recursion*.

*Guarded recursion* is recursion inside `[ guard brackets ]`.  Here's a grammar to parse `{"arrays", "that", {"look", "like"}, "this"}`:

```
element = array | string
array = [ '{' element (',' element)* '}' ]
```

The symbols just inside the brackets — `'{'` and `'}'` here — are the *begin* and *end tokens* of the guard bracket.  Begin and end tokens can't appear anywhere else in the grammar except as other begin and end tokens.  This is what it means to be *visibly pushdown*: all recursion is explicitly delineated by special symbols.

*Expression recursion* lets you define unary and binary operators using the `.operators` keyword:

```
expression =
  identifier | number | parens : value
 .operators prefix
  '-' : negate
 .operators infix left
  '+' : add
  '-' : subtract
parens = [ '(' expression ')' ]
```

Operators in the same `.operators` clause have the same precedence level; clauses nearer the top of the list are higher in precedence.

To learn more, check out the [grammar reference](docs/grammar-reference.md).

## reasons to use it

* Bluebird's parsing model can be understood without talking about parser states, backtracking, lookahead, or any other implementation details.
* Seeing ambiguities in terms of the parsed language makes understanding and removing them much easier.
* Bluebird's interpreter mode lets you design, debug, and test your grammar without writing any code at all.
* Bluebird's parse tree output is clear and beautiful.  Parsed ranges are shown as annotations on the input text instead of as abstract trees.  Color is used to reinforce the connection between parse tree nodes and the text they contain.
* In the generated parser, bluebird automatically creates compressed parse trees for you and provides an API for iterating and unpacking them.  This lets you focus on your application instead of on parse-tree bureaucracy.  The built-in tokenizer also reduces the code you have to write.
* The generated parser is a "single-file library" which is easy to include in your C project.

## reasons not to use it

* Bluebird is new and hasn't been used for anything substantial besides its own parser.
* As grammars get larger and more complex, the generated .h file can become quite big—tens or hundreds of thousands of lines.  This isn't a fundamental limitation of the technique, just of the current implementation (which uses precomputed DFAs).
* Bluebird only parses visibly-pushdown languages, and the language you want to parse may not be one of these.
* The performance of the generated parser is decent, but I haven't done a lot of benchmarking or optimization—other more mature parsers are likely to give you better throughput.
* The generated parser is a C header file, so if you're not using C, this may not be very useful.
* If you need more kinds of tokens than the default tokenizer can provide, custom tokenizers aren't supported at the moment.