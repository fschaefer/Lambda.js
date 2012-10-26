![Lambda.js Logo](https://github.com/fschaefer/Lambda.js/raw/master/misc/Lambda.js.png)

## What is it?

Lambda.js are string based lambdas for Node.js and the browser, that allow strings such as 
`'x+1'` and `'x -> x+1'` to be used as functions.

## Origin

This is a spin-off project of 
[Functional](https://github.com/osteele/functional-javascript)'s (a library for 
functional programming in JavaScript) 
[to-function.js](https://github.com/osteele/functional-javascript/blob/master/to-function.js) by [Oliver Steele](http://osteele.com/).

Most of the source code and documentation is written by Oliver Steele and
contributers, so all credit goes to them. This project is just here to avoid
using the full library when only lambdas are needed (they play very
well with JavaScript 1.6's `every()`, `filter()`, `forEach()` etc). It also
detaches the lambda function from `String.prototype` to avoid cluttering it
with non-standard functions.

## Usage

The only exported function is `lambda(expression)` which takes the "string 
based lambda" as argument and turns that expression into a `Function` that 
returns the value of the expression:

    var square = lambda('x*x');

    console.log(square.toString());
    /*  =>
        function anonymous(x) {
            return x * x;
        }
    */

If the string contains a `->`, this separates the parameters from the body:

    lambda('x -> x + 1')(1); // => 2

    lambda('x y -> x + 2*y')(1, 2); // => 5

    lambda('x, y -> x + 2*y')(1, 2); // => 5

Otherwise, if the string contains a `_`, this is the only parameter:

    lambda('_ + 1')(1); // => 2

If the string begins or ends with an operator or relation, prepend or append a 
parameter (The documentation refers to this type of string as a `section`):

    lambda('/2')(4); // => 2

    lambda('2/')(4); // => 0.5

    lambda('/')(2, 4); // => 0.5

Sections can end, but not begin with, `-`. (This is to avoid interpreting e.g. 
`-2*x` as a section). On the other hand, a string that either begins or ends 
with `/` is a section, so an expression that begins or ends with a regular 
expression literal needs an explicit parameter.

Otherwise, each variable name is an implicit parameter:

    lambda('x + 1')(1); // => 2

    lambda('x + 2*y')(1, 2); // => 5

    lambda('y + 2*x')(1, 2); // => 5

Implicit parameter detection ignores strings literals, variable names that 
start with capitals and identifiers that precede `:` or follow `.`:

    ['probable', 'possible'].map(lambda('"im" + root')); // => ["improbable", "impossible"]

    lambda('Math.cos(angle)')(Math.PI); // => -1

    lambda('point.x')({x:1, y:2}); // => 1

    lambda('({x:1, y:2})[key]')('x'); // => 1

Implicit parameter detection mistakenly looks inside regular expression 
literals for variable names. It also doesn't know to ignore JavaScript keywords 
and bound variables. (The only way you can get these last two is with a 
function literal inside the string. This is outside the intended use case for 
string lambdas.)

Use `_` (to define a unary function) or `->`, if the string contains anything 
that looks like a free variable but shouldn't be used as a parameter, or to 
specify parameters that are ordered differently from their first occurrence in 
the string.

Chain `->`s to create a function in uncurried form:

    lambda('x -> y -> x + 2*y')(1)(2); // => 5

    lambda('x -> y -> z -> x + 2*y+3*z')(1)(2)(3) // => 14

