# LilGo

LilGo is a complier and bytecode VM for the LilGo programming language.
LilGo is a subset of the [go programming language](https://go.dev/) and contains basic syntax like if statments and for loops.
You can find the formal grammer for the LilGo language below.

This project was inspired by the [Tiny-C language](https://gist.github.com/seanjensengrey/874a1dcdb7b40407ac916dd2090051a4) created by Marc Feeley
Currently the LilGo comes in a *290* lines of code total.
```sh
$ cloc main.go
       1 text file.
       1 unique file.                              
       0 files ignored.

github.com/AlDanial/cloc v 1.98  T=0.01 s (155.3 files/s, 54814.3 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Go                               1             27             36            290
-------------------------------------------------------------------------------
```
this includes a `lexer`, `parser`, `complier` and `bytecode VM`.

# Quick Start

You will need [go](https://go.dev/learn/) installed to use LilGo.

Clone this repo using git.
```sh
$ git clone git@github.com:bjatkin/LilGo.git
```

Build the complier using the `go` command.
```sh
$ go build .
```

This should produce a binary named `lilgo`.
This both the complier as well as the bytecode VM for the LilGo language.
To test out the language for yourself save the following code to a file named `hello.lil`
```go
a = 10
if a == 10 {
	b = 20
}
```

You can complie this code by running:
```sh
$ ./lilgo build hello.lil
```

`build` is the command to complie your code.
It will result in a binary file named `hello` written to the same directory as the `hello.lil` file.
You can then run this file by running:
```sh
$ ./lilgo run hello
a = 10
b = 20
```

Your code will run and all non-zero variables will be printed out to the console.
You can learn more about the langage features by in the [Learn LilGo](#learn-lilgo) section.
You can also view the code examples in the [examples](https://github.com/bjatkin/LilGo/blob/main/examples) folder.
Also consider checking out the [LilGo formal Grammer](#lilgo-specification) to check out a more formal spec for the language.

# Learn LilGo

The standard file extension for a LilGo source file is `.lil`.
Also note that the complier prioritizes size over all else, meaning error messages are pretty bad.
If you're not sure why you're getting errors you may need to dig into the code yourself to track down the issue.

### Comments

LilGo supports comment lines starting with the `//` characters.
Any line that starts with a double slash will be ignored by the complier.
You can also add `//` comments to the end of lines.
Block comments are not supported.

### Types

LilGo only supports 1 type which is uint8.
This means all values in LilGo must be in the range 0 to 255.

### Keywords

**if**
```go
if a == b {
	...
}

if (a + 5) < 10 {
	...
}
```

The **if** keyword *MUST* be followed by an expression and a code block `{\n ... }\n`.
LilGo only support 1 type (uin8) so intead of checking for a `true` or `false` bool value, if check for a `0` value.
If the if expression results in any value other than a `0` the body will be run.

*Note: curly braces in LilGo must be followed by new line tokens to be valid `{\n ... }\n`*

**if ... else**
```go
if a == b {
	...
} else {
	...
}
```

The **if** keyword can optionally include an **else** code block.
If the expression resolves to a `0`, the else block will be run, rather than the if block.

*Note: curly braces in LilGo must be followed by new line tokens to be valid `{\n ... }\n`*

**for**
```go
for i = 0; i < 10; i = i + 1 {
	...
}

i = 0
for ; i < 10; {
	i = i + 1
	...
}
```

The **for** keyword is LilGo's looping construct.
The first and last expressions in the for loop are optinal and can be omitted if needed.
However, the `test` expression must be included.
This is because LilGo has no **break** keyword, meaning **for** loops that are missing a `test` expression would never terminate

*Note: curly braces in LilGo must be followed by new line tokens to be valid `{\n ... }\n`*

### Variables

There are exactly 26 valid variables in LilGo.
They are the lowercase letters `a` through `z`.
They are all declared globally by the run time so there is no need to declare them before using them.
Variables can be set using the **=** operator

```go
a = 10
b = a
c = (a + b) - 5
```

When your program has run to completion, all non-zero variables will be printed out the the console.
This is how you can get output from your programs.

### Equality Tests

There are 4 operators which can be used to test values for equality.
They are **==**, **<**, **>**, and **!=**.
These work as you would expect with the difference that they return a `uint8` rather than a `bool`.
This is because LilGo only supports the `uint8` data type.
If the operator would return `true` then it returns `1` otherwise it returns `0`
```go
a = 5
b = 10

5 < 10 // results in 1
5 > 10 // results in 0
5 == 10 // results in 0
5 != 10 // results in 1
```

### Arithmatic

LilGo supports 2 arithmetic operators **+** and **-**.
They opperate on `uint8` values which are the only values supported by LilGo.
**+** adds two values together, overflowing when the value passes 255.
**-** subtracts two values from one another, underflowing when the value passes 0.
Numeric literals such as `1000` will also overflow.
However, negative numeric literals like `-10` are not valid in LilGo as it does not support the uniary negation operator.

# LilGo Specification

The formal grammar for `LilGo` is described bellow using [Extended Backus-Naur form(EBNF)](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form):

```
<script> ::= {<stmt>}
<stmt> ::= "if" <expr> <block_stmt>
		 | "if" <expr> <block_stmt> "else" <block_stmt>
	     | "for" [<var_stmt>] ";" <expr> ";" [<var_stmt>] <block_stmt>
		 | <var_stmt> "\n"
<block_stmt> ::= "{\n" {<stmt>} "}\n"
<var_stmt> ::= <id> "=" <expr> | <expr>
<expr> := <id> "=" <expr>
<expr> ::= <sum_expr>
		 | <expr> "<" <expr>
		 | <expr> ">" <expr>
		 | <expr> "==" <expr>
		 | <expr> "!=" <expr>
<sum_expr> ::= <primary> "+" <sum_expr>
			 | <primary> "-" <sum_expr>
			 | <primary>
<primary> ::= <uint8> | <var> | "(" <expr> ")"
<var> ::= "a" | ... | "z"
<uint8> ::= 0 | ... | 2^8
```

The LilGo bytecode VM *MUST* allocate space for all 26 possible variables.
All 26 values *MUST* be initalized to zero before executing code.
All values in LilGo are 8 bit values ranging from 0 to 255.
