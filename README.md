# Lego: Kernel lang proposal

Influences: Ioke, Lisp, Ruby, Smalltalk

Authors: José Valim, Yehuda Katz

This proposal outlines the grammar for the Lego kernel language. The Lego kernel language is a minimal specification that provides a very simple but elegant syntax in order to work as building blocks for other languages. It provides operator and container tables in order to allow different languages built on top of Lego to provide significantly different features and syntax.

# Syntax

The Lego language is made of three elements: macros, functions and key-value args (besides few literals). Here is how we can define a function that sums two variables:

    def(sum(a, b), do: +(a, b))

In this example, we are calling the macro `def` passing two arguments: a function call expressions (`sum` with values `a` and `b`) and a key-value argument `do` with an expression. We may expand the `do` into a series of expressions. For that, we use additional parenthesis:

    def(math(a, b), do: (
      =(c, +(a, b))
      *(c, *(a, b))
    ))

For instance, `if/else` clauses could be implemented as:

    if(some_variable, do: (
      invoke_some_function()
    ), else: (
      done()
    ))

Similarly, a function that sums two numbers is defined as:

    fn(a, b, do: (
      +(a, b)
    ))

Finally, Lego also allows `;` to separate several expressions in the same line:

    def(math(a, b), do: (=(c, +(a, b)); *(c, *(a, b))))

This is the basic specification of the language. So far, there are just five characters reserved by the language: `,` `(` `)` `:` `;`

## Getting rid of the parenthesis

The Lego kernel language provides several conveniences to get rid of parenthesis:

* Operators;
* Optional parenthesis;
* Key-value blocks.

### Operators

Lego provides operators and an operator table. For instance, the following example (already shown above):

    def(math(a, b), do: (
      =(c, +(a, b))
      *(c, *(a, b))
    ))

Could be rewritten as:

    def(math(a, b), do: (
      c = a + b
      c * a * b
    ))

Lego operator table will be able to handle unary, binary and ternary operators. All entries in the operator table have a precedence associated to them. A language built on top of Lego may optionally expose the operator table to developers so they can add their own operators at runtime (similar to the io language).

Notice that an operator is not limited to only symbols. If a language wishes, `div` could be defined as a binary operator in the operator table. A practical example are guards, which could be implemented as:

    def(math(a, b) when is_number(a) and is_number(b), do: (
      c = a + b
      c * a * b
    ))

The example above could be implemented by defining `when` as a binary operator and assigning a low precedence to it.

### Optional parenthesis on macro/function calls

The second convenience provided by Lego are optional parenthesis on macro/function calls. Our `math` example could now be refactored to:

    def math(a, b), do: (
      c = a + b
      c * a * b
    )

#### Solving optional parenthesis and operators ambiguity

If an operator can be used on both unary and binary forms and the language also allows optional parenthesis, the language will have some ambiguity. Here are some non-ambiguous examples:

    some_function() + 1
    some_function(+1)

If we remove parenthesis, the parser no longer knows how to handle both expressions without adding an special case:

    some_function + 1
    some_function +1
    some_function+1

To solve this problem, this specification defines that those three forms above translate respectively to:

    some_function() + 1
    some_function(+1)
    some_function() + 1

In general, white-space should be ignored by Lego implementations, except in the scenario above where white-space must be used to remove ambiguity. This follows the same rules as the Ruby programming language parser.

Finally, it is important to notice that, once an operator is added to the operator table, operator function calls require explicit parenthesis. For instance, both of these expressions are valid:

    +(1, 2)
    1 + 2

But `+ 1, 2` isn't.

### Key-value blocks

The third convenience provided by Lego are key-value blocks. Key-value blocks encapsulates the common patterns in the language and allow us to provide key-value args with expressions without a need to use parenthesis. With this feature, our `math` example could be rewritten as:

    def math(a, b) do
      c = a + b
      c * a * b
    end

Everything marked by the newly introduced `do`/`end` keywords is passed as value in the `do:` key-value argument. Key-value blocks become more useful when we add other keywords to the block. For example, our `if`/`else` example already shown above:

    if(some_variable, do: (
      invoke_some_function()
    ), else: (
      done()
    ))

Could now be rewritten as:

    if some_variable do
      invoke_some_function
    else:
      done
    end

This is similar to Ruby blocks. In fact, if we want to insert parenthesis, they would be inserted as follow:

    if(some_variable) do
      invoke_some_function
    else:
      done
    end

Key-value blocks works the same as key-value args with one important difference. Key-value blocks allow multiple values for the same key. This is convenient to implement `case`/`when` (also known as `switch`/`case` in some languages):

    case some_var do
    when: 0
    when: 1
      puts "is zero or one"
    when: 2
      puts "is two"
    when:
      puts "none of above"
    end

Finally, key-value blocks also have an alternative syntax as `->`/`end`. This alternate syntax is important because, while `do`/`end` binds to the farthest function call, `->`/`end` binds to the closest. For instance:

    foo bar do
      some_call
    end

It is the same as:

    foo(bar) do
      some_call
    end

However:

    foo bar ->
      some_call
    end

Which is the same as:

    foo(bar ->
      some_call
    end)

This difference is crucial when working with functions. For example:

    System.add_callback fn(state) ->
      print state
    end

Using `do` instead `->` would likely cause an error because no implementation would be bound to the function.

## Data types

This section specifies a syntax mechanism for the implementation of custom data types besides literals.

### Literals

The following are literals in Lego:

    :atom
    1
    2.0
    100_000

### Containers

In order to support the definition of other data types, Lego provides the concept of containers. A container uses two delimiters to mark the contained elements. For instance, here is how a list or an array could be defined (as in the examples above):

    [1,2,3]

Which internally is identified and translated to:

    [](1, 2, 3)

Another data structure could similarly be defined  as:

    { 1, 2, 3 }

Which then translates to:

    {}(1, 2, 3)

And so forth. Since those are simply a macro/function call, we could implement Ruby like hashes using keyword args:

    { a: foo, b: bar }
    {}(a: foo, b: bar)

## Parenthesis applicability

In Lego, parenthesis may apply to any expression although their behavior may be or not supported by the language. For instance, a language could treat all expressions as functions, therefore all those should be valid syntactically:

    1(2)
    [1,2,3](0)

A language may also allow parenthesis to be applied to an special operator. For instance, imagine a implementation where `.` is a binary operator:

    foo.bar(1, 2)

This example would translate to the form below, which in a language like Ruby would mean method dispatching:

    .(foo, bar)(1, 2)

Besides, Lego also supports parenthesis to be applied as in the expression below:

    foo.(1,2)

Such example would translate to:

    .(foo)(1,2)

Which has similar translation as a unary operator.

## Wrapping up

So far, we have detailed the syntax of the language and introduced conveniences. With the macro mechanism, we were able to avoid defining several keywords and with a few syntax additions, the language looks pleasant and flexible to work with.

The keywords are limited to: `,` `(` `)` `:` `;` `do` `end` `{` `}`

# BNF grammar sample

TODO

## Invalid syntax examples

In this section, we are going to describe some examples that are invalid code according to this specification.

### Wrong usage of parenthesis

In Lego, parenthesis are used for grouping expressions or doing calls (read Parenthesis applicability section above). Any other usage of parenthesis is invalid. For example, this common syntax for functions is invalid:

    `(a, b) -> (a + b)`

The reason is that `(a, b)` is supposed to apply to an expression, but it actually doesn't apply to anything.