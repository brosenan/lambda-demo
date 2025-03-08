# Lambda: An Example Language

In this tutorial we will define a complete language from the ground up using
[y0](https://github.com/brosenan/y0/), providing tool support for it.

The language we choose to define is `lambda`, a made-up language that, as its
name suggests, is model after the [Lambda
Calculus](https://en.wikipedia.org/wiki/Lambda_calculus). Unlike most functional
programming languages, which are also modeled after it, `lambda` is almost an
exact replica of the Lambda Calculus, adding almost nothing to it.

We choose to implement such a language because on the one hand, it is simple,
but on the other hand, requires many of $y_0$'s features.

## The `lambda` Language

Below is a simple `lambda` program, adopted from [the wikipedia article on the
Lambda Calculus](https://en.wikipedia.org/wiki/Lambda_calculus).

```haskell
-- The identity function
id = \x. x;

-- Church Numerals
zero = \f.\x.x;
one = \f.\x.f x;
two = \f.\x.f (f x);

-- Arithmetic
PLUS = \m.\n.\f.\x.m f (n f x);
MULT = \m.\n.m (PLUS n) zero;

-- Logic
TRUE := \x.\y.x;
FALSE := \x.\y.y;
AND := \p.\q.p q p;
OR := \p.\q.p p q;
NOT := \p.p FALSE TRUE;
IFTHENELSE := \p.\a.\b.p a b;
```

As can be seen in the example above, a `lambda` program is made of _definitions_
of the form `var = expr;`, where `var` is an identifier and `expr` is a lambda
expression. A lambda expression can be either a _variable_, a _lambda
abstraction_ of the form `\var.expr` (`\` being an ASCII representation of the
$\lambda$ symbol), and _function application_ of the form `f x`, where both `f`
and `x` are variables.

We will add a few more features to the language later, but these are the basics.

## Getting Started

Now that we have an idea as to what the `lambda` language should look like, we
can start defining it using the $y_0$ language and the tools built around it.

We will work in two steps. First, we will define the `lambda` language and test
it using its spec, which is this very `.md` file. Then, we will show how to run
a language server based on this definition, to provide tool support for the
language.

For the first part, we will use
[y0.jar](https://github.com/brosenan/y0/blob/main/lsp/bin/y0.jar), an executable
JAR file containing the $y_0$ CLI tool. Please download this file by clicking
"Raw" on the right-hand side of the page, and place it in a new (empty) project
directory.

Now, also place a copy of this file (`lambda-spec.md`) in that same directory.

Make sure you have a terminal open within that directory, and that you have some
recent version of `java` installed and in your `PATH`.

To check that everything's set, run:

```sh
$ java -jar y0.jar --help
WARNING: assert already refers to: #'clojure.core/assert in namespace: y0.core, being replaced by: #'y0.core/assert
WARNING: with-meta already refers to: #'clojure.core/with-meta in namespace: y0.core, being replaced by: #'y0.core/with-meta
WARNING: import already refers to: #'clojure.core/import in namespace: y0.core, being replaced by: #'y0.core/import
WARNING: = already refers to: #'clojure.core/= in namespace: y0.core, being replaced by: #'y0.core/=
WARNING: replace already refers to: #'clojure.core/replace in namespace: y0.explanation, being replaced by: #'clojure.string/replace
  -p, --path PATH                    []  Add to Y0_PATH
  -c, --language-config CONFIG_PATH  {}  Path to the language configuration file
  -l, --language LANG                y0  The language in which the given module names are written
  -s, --language-spec                    If this is set, then the positional arguments are expected to be paths to markdown files, to be evaluated as language-specs
  -u, --update-language-spec             Same as -s, but will also update the markdown files with current statuses
  -h, --help
```

Feel free to ignore the warnings, which come from the Clojure implementation.
The output provides the usage for the command line, which we will use in the
next sections.

## The Language Config

A language definition starts with a [language
config](https://github.com/brosenan/y0/blob/main/doc/config.md). This
configuration file defines the behavior of a given language, starting with its
syntax, continuing to the behavior of its module system and ending with a
reference to a $y_0$ module which defines its semantics.

For `lambda`, we will write the following in `lang-conf.edn`.

```clojure
{"lambda"
 {;; Use an Instaparser
  :parser :insta
  ;; The grammar with a layout section
  :grammar "TBD"
  ;; The keyword (or keywords) representing an identifier in the grammar
  :identifier-kws :identifier
  ;; The keyword representing a dependency module name in the grammar
  :dependency-kw :dep
  ;; The resolver is based on a prefix list
  :resolver :prefix-list
  ;; The path relative to the prefixes is based on dots in the module name
  ;; (e.g., a.b.c => a/b/c.ext)
  :relative-path-resolution :dots
  ;; The file extension for the source files
  :file-ext "lambda"
  ;; The modules that define the language's semantics
  :y0-modules ["lambda"]
  ;; Read files using Clojure's slurp function.
  :reader :slurp
  ;; Stringify by extracting text from the source file
  :expr-stringifier :extract-text
  ;; Decorate the tree
  :decorate :true
  ;; Language stylesheet
  :stylesheet [{}]}}
```

This file is written in the [EDN](https://github.com/edn-format/edn) format, and
contains a [map](https://github.com/edn-format/edn?tab=readme-ov-file#maps) from
a name of a language to a map of attributes for this language.

We will not dive into explaining all of them (see the comments for some
explanation), but rather focus on key ones.

1. `:parser`: This is where we select the parsing algorithm to be used. `:insta`
   represents the [instaparse](https://github.com/Engelberg/instaparse) Clojure
   library. Once selected, `:grammar` is expected to contain a valid instaparse
   grammar.
2. `file-ext` contains the file extension for `lambda` files.
3. `:y0-modules` contains a list of module names (without the `.y0` extensions)
   that will contain the semantic definition of the language.
4. `:stylesheet` is _not_ a part of the language definition per-se, but rather a
   CSS-like collection of rules that define attributes for parse-tree nodes,
   required to provide some LSP services. But we are way ahead of ourselves
   here.

## Initial Language

For this definition to be valid we need to do two things: replace the `TBD` with
an acual (initial) grammar, and add a `.y0` semantic definition file.

### The Initial Grammar

Replace the `:grammar` in `lang-conf.edn` with the following:

```clojure
  :grammar "compilation_unit = definition*
            
            definition = identifier <'='> expr <';'>
            
            expr = identifier
            
            identifier = #'[a-zA-Z_][a-zA-Z_0-9]*'
            
            --layout--
            layout = #'\\s'+"
```

This grammar defines a `compilation_unit` (the contents of a single source file)
as a sequence of `definition`s. Each `definition` is an `identifier`, followed
by a `=`, followed by an `expr`ession, followed by a `;`.

An `expr` is currently just an `identifier`. We will add more options as we grow
our language.

Finally. an `identifier` is defined using a regular expression.

Below that, still withing the `:grammar` string, the `--layout--` separator
separates between the "main" grammar, defining the language in terms of
"high-level" tokens, and the `layout`, which defines characters that can appear
between such tokens, such as white-space (in this example) and comments (which
will be added later).

The `<>` markers around `=` and `;` are used to omit these symbols from the
generated parse-tree. We will generally omit any token that does not add value
to the parse tree.

### Initial Semantic Definition

Create a `lambda.y0` file in the root directory of your project, with the
following contents:

```clojure
(ns lambda)
```

Now we can test the definition of this new language. We do this by adding an
example to the spec, which this file doubles as.

At the beginning of a spec file, before providing any examples, we need to
associate this spec with a language in the language config. The following line
does the trick:

Language: `lambda`

An example consists of a code block, followed directly by a `status` block.

Our first example will contain a single definition, which is the only definition
we can generate with our current grammar.

```haskell
foo = bar;
```
```status
ERROR: bar is not a valid lambda expression in foo = bar;
```

Now, we can test our language definition against this example, by running:

```sh
$ java -jar y0.jar -c lang-conf.edn -p . -s lambda-spec.md 
WARNING: assert already refers to: #'clojure.core/assert in namespace: y0.core, being replaced by: #'y0.core/assert
# ...
nil
lambda-spec.md:206: Error: The wrong error was reported: No rules are defined to translate statement [:definition foo [:expr ...]] and therefore it does not have any meaning
lambda-spec.md:206: Note: [:definition example/foo [:expr example/bar]]
1 Failed 
```

Let's break it down. The command line calls `y0.jar`, giving it the language
config and the spec file. `-p .` tells it to find `.y0` files in the current
directory.

The error we got tells us that, while we expected one error to be reported from
our program, we actually received a different one. We expected a message telling
us that `bar` is not a valid lambda expression, but instead got something about
`:definition` not having rules to translate it.

The `:definition` object is the parse-tree object that corresponds to the
`definition` grammar rule. To fix this error, we need to create a [translation
rule](https://github.com/brosenan/y0/blob/main/doc/statements.md#translation-rules)
to transform it into something else.

Let's start simple, by adding a translation rule that translates it into nothing
in `lambda.y0`:

```clojure
(all [v x]
     [:definition v x] =>)
```

This rule states that for all `v` and `x`, the term `[:definition v x]` (which
is the result of parsing `v = x;`)... means nothing.

Now we get a different failure:

```sh
$ java -jar y0.jar -c lang-conf.edn -p . -s lambda-spec.md 
# ...
Error: The example should have produced an error, but did not
1 Failed 
```

The message states that despite expecting an error, we are actually not getting
one. This is due to the translation rule not doing anything and thus not
checking the validity of the expression.

To do just that, we wish to add an `assert` block, to test that `x` is a valid
lambda expression.

```clojure
(all [v x]
     [:definition v x] =>
     (assert (lambda-expr x)))
```

Now, our translation rule translates a definition into an assertion that the
right-hand side is a `lambda-expr`. Running it will tell us that we haven't
defined `lambda-expr` yet.

```sh
/home/boaz/clj/lambda-demo/./lambda.y0:5: Error: The wrong error was reported: Undefined predicate lambda-expr with arity 1
/home/boaz/clj/lambda-demo/./lambda.y0:5: Note: /home/boaz/clj/lambda-demo/./lambda.y0/lambda-expr
1 Failed 
```

To fix this, we define a base case for `lambda-expr`.

```clojure
(all [x]
     (lambda-expr x ! x "is not a valid lambda expression"))
```

This rule states that for all `x`, testing whether `x` is a lambda expression
fails, providing the specified error message.

This is enough for our first example to pass.

```sh
$ java -jar y0.jar -c lang-conf.edn -p . -s lambda-spec.md 
# ...
1 Succeeded
```

### Step 1 Complete

At this point you should have the files `lang-conf.edn` and `lambda.y0` as in
the [step1](step1/) directory.
