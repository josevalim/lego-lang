# Lego: Kernel lang proposal

Influences: Ioke, Lisp, Ruby, Smalltalk

Authors: José Valim, Yehuda Katz

This proposal outlines the grammar for the Lego kernel language. The Lego kernel language is a minimal specification that provides a very simple but elegant syntax in order to work as building blocks for other languages. It provides operator and container tables in order to allow different languages built on top of Lego to provide significantly different features and syntax.

## Syntax

The Lego language is made of three elements: macros, functions and key-value args (besides few literals). Here is how we can define a method that sums two variables:

    def(sum: [a, b], do: +(a, b))

In this example, we are calling the macro `def` passing two key-value args, `sum` with value [a, b] and `do` with an expression. We may expand the `do` into a series of expressions. For that, we use additional parenthesis:

    def(math: [a, b], do: (
      =(c, a + b)
      *(c, *(a, b))
    ))

For instance, `if/else` clauses could be implemented as:

    if(some_variable, do: (
      invoke_some_method()
    ), else: (
      done()
    ))

Similarly, a function that sums to numbers is defined as:

    fn(a, b, do: (
      +(a, b)
    ))

Finally, Lego also allows `;` to separate several expressions in the same line:

    def(math: [a, b], do: (=(c, a + b); *(c, *(a, b))))

This is the basic specification of the language So far, there are just four characters reserved by the language: `,`, `(`, `)`, `:` and `;`.

## Getting rid of the parenthesis

The Lego kernel language provides several conveniences to get rid of parenthesis. Let's take a look.

### Operators

Lego will provide operators and an operator table. For instance, the following example (already shown above):

    def(math: [a, b], do: (
      =(c, a + b)
      *(c, *(a, b))
    ))

Could be rewritten as:

    def(math: [a, b], do: (
      c = a + b
      c * a * b
    ))

Lego operator table will be able to handle unary, binary and ternary operators. All entries in the operator table have a precedence associated to them. A language built on top of Lego may optionally expose the operator table to developers so they can add their own operators at runtime (similar to the io language).

Notice that an operator is not limited to only symbols. If a language wishes, `div` could be defined as a binary operator in the operator table. An example would be to implement guards in the syntax proposed so far:

    def(math: [a, b] when is_number(a) and is_number(b), do: (
      c = a + b
      c * a * b
    ))

The example above could be implemented by defining `when` as operator and assigning a high precedence to it.

### Optional parenthesis on macro/function calls

The second convenience provided by Lego are optional parenthesis on macro/function calls. Our `math` example could now be refactored to:

    def math: [a, b], do: (
      c = a + b
      c * a * b
    )

### Key-value blocks

The third convenience provided by Lego are key-value blocks. Key-value blocks encapsulates the common patterns in the language and allow us to provide key-value args with expressions without a need to use parenthesis. With this feature, our `math` example could be rewritten as:

    def math: [a, b] do
      c = a + b
      c * a * b
    end

Everything marked by the newly introduced `do`/`end` keywords is passed as value in the `do:` key-value argument. This is similar to Ruby blocks. In fact, if we want to insert parenthesis, they would be inserted as follow:

    def(math: [a, b]) do
      c = a + b
      c * a * b
    end

Key-value blocks become more useful when we add other keywords to the block. For example, our `if`/`else` example already shown above:

    if(some_variable, do: (
      invoke_some_method()
    ), else: (
      done()
    ))

Could now be rewritten as:

    if some_variable do
      invoke_some_method
    else:
      done
    end

There two other features provided syntactically by key-value blocks. The first one is the ability to handle empty expressions. The second allows them to work as accumulators to duplicated key blocks. This is convenient to implement `case`/`match` (also known as `switch`/`case` and `case`/`when` in other languages):

    case some_var do
    match: 0
    match: 1
      puts "is zero or one"
    match: 2
      puts "is two"
    else:
      puts "none of above"
    end

Internally, this is translated to:

    case(some_var, do: (), match: [(0), (1; puts "is one"), (2; puts "is two")], else: (puts "none of above"))

When used in method calls without parenthesis, `do`/`end` always applies to the furthest method call. For instance:

    foo bar do
      some_call
    end

It the same as:

    foo(bar) do
      some_call
    end

### Wrapping up

So far, we have detailed the kernel of the language and introduced conveniences. With the macro mechanism, we were able to avoid defining several keywords and with a few syntax additions, the language looks pleasant and flexible to work with.

The keywords are limited to: `,`, `(`, `)`, `:`, `;`, `do`, `end`, `{` and `}`.

## Literals

The following are literals in Lego:

    :atom
    1
    2.0

## Containers

In order to support the definition of other data types, Lego provides the concept of containers. A container uses two delimiters to mark the contained elements. For instance, here is how a list or an array could be defined (as in the examples above):

    [1,2,3]

Which internally is identified and translated to:

    [](1, 2, 3)

Another data structure could similarly be defined  as:

    { 1, 2, 3 }

Which then translates to:

    {}(1, 2, 3)

Since those are simply a macro/function call, we could implement Ruby like hashes using keyword args:

    { a: foo, b: bar }
    {}(a: foo, b: bar)

Notice that, even though the Lego kernel language uses curly brackets for key-value blocks, they can also be used as containers. For that, it is important to take into account the following:

    some_method { foo }

In the example above, is { foo } a key-value blocks or a container with foo? The proper answer is the former. In order to obtain the latter, explicit parenthesis would be required:

    some_method({ foo })

Lego will provide a container table where containers could be specified. Due to conflicts, a container cannot use delimiters that are also in the operators table.

## Macros

TODO

## BNF grammar sample

TODO