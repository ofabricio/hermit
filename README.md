![stage](https://img.shields.io/static/v1?label=stage&message=concept&color=red)

# Hermit

Hermit programming language. This is a programming language concept. Just random ideas for a language.
Whether feasible, good or bad I don't know.

## Key features

- Compiled.
- Type inference.
- Strong and static type system.
- Generics.
- Sum types.
- No global state.
- No `if` keyword.
- Pattern matching.
- Destructuring assignment.
- Syntax sugar or preprocessor.
- Function currying / partial application.
- Functions can return `Some`, `None` or `Err` values.
- Things can be tagged with `: name` notation. This is like declaring a variable.

## Hermit code examples

```c
tokenize (input str) E | [(str, str)] : tokens {
    for input = input[pos:] {
        pos, type = $.match_len(input) {{
            re_nl         ! 'NL'
            re_whitespace ! continue
            re_comment    ! continue
            re_number     ! 'Number'
            re_alphanum   ! 'Ident'
            re_symbols    ! 'Symbol'
            else          ! ret E('Unexpected token {input}')
        }}
        token = input[:pos]
        tokens += (type, token)
    }
}
```

```c
fib (n i32) i32 {
    n <= 1 ! ret n
    ret fib(n - 1) + fib(n - 2)
}
```

```c
min (a, b i32) i32 {
    ret a < b {{ a b }}
}
```

## Return value handling / Error handling

```c
examples() NE {

    get_some() {
        print('Handle truthy value {S}')
    }

    get_none() N {
        print('Handle none')
    }

    get_error() E {
        print('Handle error {E}')
    }

    msg = get_some_none_error() {{
        S ! 'Got some {S}'
        E ! 'Got err {E}'
        N ! 'Got none'
    }}

    get_some() get_some() : a b {
        print('Got {a} and {b}')
    }

    get_none() | get_some() : a a {
        print('Got {a}')
    }

    get_error()? // Return the error.

}
```

## Control statement

Conditions are handled with `{ }`. There is no `if` keyword. Values are falsy or truthy.

```c
true {
    print('Truthy condition is handled here.')
}
```

```c
false {
    print('This is not print.')
} else {
    print('Falsy condition is handled here.')
}
```

```c
a = true { 2 } else { 3 }
b = true {{ 2 3 }}
print(a)
print(b)
```

Logical AND/OR with `&` and `|`.

```c
false | true {
    print('Only one is true.')
}
```

```c
true & true {
    print('Both are true.')
}
```

It's possible to omit the symbol for AND.

```c
true true {
    print('Both are true.')
}
```

Capture values with tags.

```c
get_some() get_some() : a b {
    print('Values are {a} {b}')
}
```

```c
get_none() | get_some() : a a {
    print('Got {a}')
}
```

Shorter form conditional.

```c
true ! ret 1
true { ret 1 } // Same as above.

false ! print('No print')
```

## Pattern matching

```c
v = 'hello' {{
    'hello' ! 'String is hello'
    'hi'    ! 'String is hi'
    else    ! 'String is something else'
}}
print(v) // String is hello
```

For multiline code:

```c
fruit {{
    apple {
        print(1)
        print(2)
    }
    else {}
}}
```

When the argument is a boolean there's no need to provide the two matching options,
you can simply return a value in each line:

```c
a = true {{
    3
    7
}}
b = false {{
    3
    7
}}
print(a) // 3
print(b) // 7
```

With the `$` symbol the compiler replaces the symbol with the first column of the pattern matching.
So this:

```c
length, type = re.match_len($, 'eax') {{
    '(add|mov)' ! 'Instruction'
    '(eax|ebx)' ! 'Register'
    else        ! 'Unknown'
}}
print('{type} {length}') // Register 3
```

Is syntax sugar for this:

```c
type = re.match_len('(add|mov)', 'eax') : length {
    'Instruction'
} else re.match_len('(eax|ebx)', 'eax') : length {
    'Register'
} else {
    'Unknown'
}
print('{type} {length}') // Register 3
```

On each condition the return value is assigned to `length`.
The `type` variable holds the return value of the body.

## Destructuring assignment

```c
rect = Rectangle(x: 1, y: 2, width: 3, height: 4)

width, height = rect.$
```

## Syntax sugar or preprocessor

```c
pt = Point(x: 1, y: 2)

// This:
pt.$(x y) = rect.$(x y)

// Is syntax sugar for:
pt.x = rect.x
pt.y = rect.y
```

```c
// To create a Point out of a Rectangle:
pt = rect.$Point(x y)

// The above is syntax sugar for:
pt = Point(x: rect.x, y: rect.y)
```

```c
// Given:
a = Vec3D(x: 1, y: 2, z: 3)
b = Vec3D(x: 4, y: 5, z: 6)

// This:
dot.$(x y z) = a * b.$(x + y + z)

// Is syntax sugar for:
dot.$(x y z) = (a.x * b.x) + (a.y * b.y) + (a.z * b.z)
```

```c
// Given:
pos = Vec2D(x: 2, y: 3)
vel = Vec2D(x: 1, y: 0)

width, height = 10, 20
block_size = 100

// This:

$(new_x new_y) = pos.$(x y) + vel.$(x y)

pos.$(x y) += $(new_x new_y) > block_size & $(new_x new_y) < $(width height) - block_size {{
    vel.$(x y)
    0
}}

// Is syntax sugar for:

new_x = pos.x + vel.x
new_y = pos.y + vel.y

pos.x += new_x > block_size & new_x < width - block_size {{
    vel.x
    0
}}
pos.y += new_y > block_size & new_y < height - block_size {{
    vel.y
    0
}}
```

```c
// This:

$(min max) (a, b i32) i32 {
    ret a $(< >) b {{ a b }}
}

// Is syntax sugar for:

min (a, b i32) i32 {
    ret a < b {{ a b }}
}

max (a, b i32) i32 {
    ret a > b {{ a b }}
}
```

## Generics

TODO

## Data Types (aka structs)

To define a struct:

```c
type Point (
    x u32
    y u32
)
```

Anonymous fields are possible:

```c
type Point (u32, u32)
```

To instantiate a struct:

```c
p = Point()
p = Point(1)
p = Point(1, 2)
p = Point(x: 1, y: 2)
p = Point() + (1,)
p = Point() + (, 2)
p = Point() + (1, 2)
p = Point() + (y: 2,)
```

Structs are created on the stack. To create them on the heap:

```c
p = &Point(1, 2)
```

To access fields:

```c
v = p.x + p.y
v = p[0] + p[1]
```

Embedding structs:

```c
type Shape ()

type Circle (
    Shape;
    radius f64
)
```

To assign functions to a struct and access its fields:

```c
Circle.area() f64 {
    ret PI * .radius ** 2
}
```

#### Anonymous structs

```c
type Player (
    name str
    prop (
        hp i32
        sp i32
    )
    tile (i32, i32)
)
```

## Sum types

```c
type Expr = Number | BinOp

type Number ()
type BinOp (a Expr, o Operator, b Expr)
```

## Enums

```c
enum Operator {
    Add,
    Sub,
    Mul,
    Div,
}
```

## APIs (aka Interfaces)

```c
api Animal {
    noise()
}
```

## Functions

Functions can return `Some`, `Err` or `None`, but they have shorter names:
`S`, `E` and `N` respectively.

A `Some` type is any value which is neither `None` or `Err`.

```c
some_or_none_or_error(a i32) i32 | NE {
    a == 0 ! ret E('arg cannot be zero')
    a == 1 ! ret N
    ret a * 10
}
```

There are a few options to handle the return value.

Pattern matching.

```c
some_or_none_or_error(2) {{
    S ! print('Got {some}')
    E ! print('Got {err}')
    N ! print('Got none')
}}
```

Propagating the error with `?`.

```c
some_or_none_or_error(0)? {{
    S ! print('Got {some}')
    N ! print('Got none')
}}
```

Handling individually.

```c
x = some_or_none_or_error(1)? N { 0 }
```

```c
x = some_or_error() E { 0 }
```

Functions don't need to return a value explicitly.
In that case the return value is either `None` or the default value for the type.

```c
some_or_none(ok bool) u32 | N {
    ok ! ret 3
}
some_or_default(ok bool) u32 {
    ok ! ret 3
}

print(some_or_none(true))     // 3
print(some_or_none(false))    // None
print(some_or_default(true))  // 3
print(some_or_default(false)) // 0
```

### Currying and Partial application

```c
add_one = add(1)
value  = add_one(2)
print(value) // 3

add(a, b u32) u32 {
    ret a + b
}
```

It is possible to give a hint to the reader where is the best place
to apply a partial application in a function by using the `,,` symbol.

```c
add_all(a, b int ,, c, d int) int {
    ret a + b + c + d
}
```

## Root dot operator

```go
main() NE {
    mongo:new("mongodb://...")
        ..connect()?
        ..collection("users") : users
        ..close()? defer
}
```

The root dot operator (`..`) accesses the value returned by the root function, ie, `mongo:new()`.
This way there is no need to assign it to a variable.
The value returned by `collection("users")` is assigned to the `users` label.
The `close()` function call is deferred so it runs only when `main()` function returns.
