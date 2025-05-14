# Templates, Hierarchies, and Reflected Structures
---
### Table of Contents
1. [Introduction](#introduction)
2. [Units](#units)
    1. [Booleans](#booleans)
    2. [Integrals](#integrals)
    3. [Rationals](#rationals)
    4. [Strings](#strings)
    5. [Structs](#structs)
3. [Hierarchies](#hierarchies)
    1. [Sequences](#sequences)
    2. [Maps](#maps)
4. [Templates](#templates)
    1. [Types](#types)
    2. [Pseudotypes](#pseudotypes)
    3. [Structs](#template-structs)
    4. [Qualifiers](#qualifiers)
    5. [Remainders](#remainders)
    6. [Collapsers](#collapsers)
    7. [Predicates](#predicates)
5. [Format](#format)
    1. [.thars](#thars)
    2. [.thars.template](#tharstemplate)
6. [Implementation](#implementation)
    1. [Grammar](#grammar)
    2. [Faithfulness](#faithfulness)
    5. [RegEx](#regex)
7. [Safety](#safety)
    1. [Structs](#safety-structs)
    2. [Templates](#safety-templates)
---

### Introduction
THaRS stands for Templates, Hierarchies, and Reflected Structures. It serves as a JSON alternative. Whereas JSON is beautifully simple, THaRS is beautifully complex.

As an alternative to JSON, THaRS is designed to expand upon it instead of a complete restructuring. This means that a faithful THaRS implementation will parse JSON, though of course a simple JSON parser could never be expected to parse THaRS.

THaRS' expansion primarily relies on two functions: templates and reflected structures, from which it gets part of its name. Templates are used to perform a compatibility check against a THaRS value. Reflected structures provide a beautiful front-end method of creating data that the back-end can use.

A faithful THaRS implementation will provide a common set of values, broken down into several categories: units and hierarchies, both of which may also be broken down into several types.

### Units
##### Booleans
`boolean`

The simplest value. Represented as `true` or `false`.

##### Integrals
`integral`

Integrals may be expressed as signed integers in base-10, or unsigned binary, octal, and hexadecimal. Integrals, when represented in base-10, are always signed, and otherwise never signed.

```
123     // Integer (base-10)
+123    // Integer (+signed)
-123    // Integer (-signed)
0b101   // Integer (binary)
0o127   // Integer (octal)
0x12a   // Integer (hexadecimal)
```

Underscores may be used (following the first and subsequent digits) to make large numbers more clear.

```
123_456
// _123456      // This is technically an identifier, not a number.
0b_1010_0011    // Underscores are permitted to be the leading digit in prefixed notations.
```

##### Rationals
`rational`

Rationals may be expressed as decimals or scientific notation.

```
12.3    // Rational
1.23e1  // Rational (scientific)
```

Like integrals, underscores may also be used.

```
3_000_000.000_000
// Although an integer may be used for the coeffecient, it's still a rational.
12_000e-3
// Underscores may also lead exponents.
13.1e_2
```

##### Strings
`string`

Strings are terminated on both ends either with a single quote (`'`) or double quote (`"`). Quotes may be used inside of strings by escaping them (`\'` or `\"`). Strings may span multiple lines, and newlines may be escaped.

```
'Hello,'
"World!"

"Quotes \"inside\" of quotes!"

// The following two strings are equivalent
"Hope you're enjoying THaRS!"
"Hope you're \
enjoying THaRS!"
```

A faithful implementation will support UTF-8 strings.

##### Structs

[!! SAFETY !!](#safety-structs)

Structs cannot be defined inside of `.thars` files, but instead through the implementation's API. Structs are declared via their name and a sequence of values for their attributes in order of declaration. Structs CANNOT be declared using their attribute names.

Consider the following struct:
```
struct Employee {
    string: "name",
    integral: "unique_id",
    rational: "salary",
    boolean: "paid",
}
```

```
Employee("John", 4193, 45_000)
```

### Hierarchies
"Hierarchies" is the collective term for sequences and maps, much like "unit" is for the aforementioned types. These types are considered hierarchies and not units for their ability to nest units and other hierarchies.

##### Sequences
`[]`

Sequences, as the name would imply, are values stored in a sequential order. The order is explicit and strictly maintained.Like I just had my perception of reality shattered
[15:06]


By default, they may be of any length and contain any types of any order, including other sequences or hierarchies.

```
[42, 91, 40, 32]
[true, 123, "Hello, world!"]
[
    "Check out my:",
    [ "nested", "sequences!" ],
    [ 1, 2, 3 ],
]
```

##### Maps
`{}`

Maps are constructed of key-value pairs. The values may be of any type, much like sequences, but the keys must all be strings. Each key must be unique. There is no guarantee or expectation of order.

```
{
    "name": "John",
    "unique_id": 4193,
    "salary": 45_000,
}
```

### Templates

[!! SAFETY !!](#safety-templates)

In truth, THaRS exists in two forms: the literal value form that is stored in `.thars` files, and the template form stored `.thars.template`. Up until now only the value form has been discussed, but this section is dedicated to the template form.

A template is not exclusively defined within a `.thars.template` file, it may be defined programmatically, but the `.thars.template` file exists nonetheless for several key reasons (see [.thars.template](#tharstemplate)).

##### Types

In a `.thars` file, values are used. In a `.thars.template` file, types are used. All THaRS values have a type, and when it comes time to match a value against a template, the type system is what powers this. A template is composed of types.

A faithful implementation has the following types:
`boolean`
`rational`
`integral`
`string`
`[]` (sequence)
`{}` (map)
`any`

Hierarchies have a special syntax. By default, they are statically sized (see: [Remainders](#remainders)) and an input value must match all of its elements/pairs exactly.

```
[boolean, number]
// MATCHES:
// [true, 5]
// [5, true]
// DOES NOT MATCH:
// [true, "5"]
// [true]
// [5]

{ "name": string, "salary": number }
// MATCHES:
// { "name": "Steve", "salary": 10_000 }
// { "salary": 10_000, "name": "Steve" }
// DOES NOT MATCH:
// { "paid": false }
// { "name": "Steve" }
// { "name": "Steve", "salary": 10_000, "paid": false }
```

Note that the order matters for sequences, but not for maps.

Maps have a special feature, that being the default value. If a default value is defined inside of a template for a key, it is not necessary to write that key inside the value file. The default value is subject to the same type and qualifier.

```
{
    "name": string,
    // The "balance" key is no longer strictly necessary to input.
    "balance": integral<0..> = 0,
}
```

##### Pseudotypes

Pseudotypes are a method of aliasing types. Consider the following template:

```
pseudo Employee = {
    "name": string,
    "salary": number,
}

[
    Employee,
    Employee,
    Employee,
]
```

The following would appropriately match against the template:

```
[
    { "name": "Steve", "salary": 120_000 },
    { "name": "Joe", "salary": 140_000 },
    { "name": "Sally", "salary": 160_000 },
]
```

Notably, pseudotypes are just that - pseudo. They act purely as aliases, and do not exist inside of the `.thars` file. This is useful for complex qualifiers (see: [Templates - Qualifiers](#qualifiers)) and matching against JSON, but otherwise consider the use of structs.

##### Structs {#template-structs}

(see: [Units - Structs](#structs))

Unlike pseudotypes, structs defined in templates do make their way into the `.thars` file. They are not aliases but concrete types. When matching  a struct instance, all its attribute parameters are matched against the struct definition's attributes. Structs are defined like so:

```
struct Employee {
    string: "name",
    number: "salary",
}
```

Structs have similar syntax to maps, but are largely different. They are more akin to a sequence, as they maintain their order. Attributes may or may not be named, but the attribute name serves no purpose to the front-end. It only exists to document and assist reflection on the back-end, but is not strictly necessary.

In order to differentiate structs from maps, attribute names appear after their types.

The following is another appropriate example of a struct:

```
// map.thars.template
pseudo Vec2i = [
    integral,
    integral,
]

struct Sample {
    string,
}

struct Map {
    Vec2i: "size",
    Sample,
}

Map
```

`Map` may then be instanced like so:

```
#"map.thars.template"
Map([2, 2], Sample("samples/data.map"))
```

All attributes must be supplied when instancing a struct.

##### Qualifiers

Sometimes a type of value may be desired, but the specifications of that value are limited. For example, a number within a range, or a string that follows a format. Qualifiers are here to save the day!

A qualifier is a set of conditions upon a type, and these conditions are called tickets. When the condition is passed, the ticket is 'issued'. For a value to match against a type that accepts a qualifier, at least one ticket must be issued.

In a faithful THaRS implementation, the following types accept a qualifier:
`integral`
`rational`
`string`
`any`

All may accept a variable number of tickets, and the following are the default qualifiers:
`integral<..>`
`rational<..>`
`string</.*/>`

`any` is unique, as its default qualifier always issues a ticket.

`integral` and `rational` only accept `..` range tickets. Range tickets are always inclusive on the lower bound, but exclusive on the upper bound by default. The upper bound may be made inclusive using the `..=` operator. Range tickets accept a left hand and a right hand, both of which are optional for the `..` exclusive range. For the `..` inclusive range, the right hand side is not optional. If it is an `..=` inclusive range, they may be equal as well, but otherwise the left hand side must always be less than the right hand side. If the left hand side is not supplied, it defaults to -INF. If the right hand side is not supplied, it defaults to +INF.

|                            | `..` | `..=` |
|----------------------------|------|-------|
| default lower-bound        | -INF | -INF  |
| default upper-bound        | +INF |       |
| bouth bounds may be equal  | No   | Yes   |

Range operands may be notated however is desired by the user. The input value will simply be tested as if it were the same type as the operand. Be wary of [floating-point accuracy errors](https://en.wikipedia.org/wiki/Floating-point_arithmetic?utm_source=chatgpt.com#Accuracy_problems).

```
5..10       // OK
5.0..10.0   // OK
5.0..10     // OK
5..10.0     // OK

5..=10      // OK
5..=5       // OK

5..         // OK
..5         // OK
..          // OK
```

`string` accepts one or more regular expressions. RegEx in THaRS is always formed the same way (see: [Implementation - Grammar](#grammar)), a regular expression pattern between two forward slashes followed by character tags. What the character tags do will depend upon the implementation's flavour of RegEx.

```
string</^[a-zA-Z_0-9]*@email\.domain$/>
string</hello/, /world/>
```

`any` accepts one or more types.

```
any<integral, rational>
any<boolean, integral>
```

##### Remainders

As previously mentioned, sequences and maps are statically sized. This is made untrue by remainders however, which indicates the sequence or map but be at least the size of the types defined within, or greater. A remainder is declared following the sequence or map using the `..` operator followed by the type which all remaining types must conform to.

```
[]..any     // Sequence of any size/type
{}..any     // Map of any size/type

[boolean]..integral
// [true, 5, 3, 2]
// [false]

{ "name": string }..any<integral, rational>
// { "name": "Mary", "lambs": 1 }
// { "name": "John", "jobs": 4_256, "salary": 1.52e+23 }
```

##### Collapsers

Collapsers go hand-in-hand with remainders. After a type has been matched against, if it is tagged to collapse via the `!` operator, it will reduce its qualifier to use the first ticket that was issued. All types that are then matched against it may only be issued the same ticket.

```
// This may either be a sequence of integrals or rationals
[]..any<integral, rational>!

// This may either be a sequence of http or https websites.
[]..string</http:\/\/\w*\..{,6}\/?/, /https:\/\/\w*\..{,6}\/?/>!
```

##### Predicates

Predicates are the method by which parts of a template file may be used in other template files. A template file will include other predicates, structs, and pseudotypes. It will NOT include the template.

A faithful implementation will allow for implicit template file extensions.

```
// core.thars.template
pseudo u8 = integral<0..256>
pseudo i8 = integral<-128..128>
```

```
// common.thars.template
predicate "core"

struct Vec2u8 { u8, u8 }
struct Vec2i8 { i8, i8 }
```

```
// Namespaces may be assigned for organization and avoiding name clashes.
predicate common = "common"

[]..any<common::Vec2u8, common::Vec2i8>!
```

### Format

##### .thars

The `.thars` file has a very simple format inspired by JSON. Apart from the template inclusion, the entire file represents a value. There may be zero or one template inclusion, and no more. There must be one value. The template inclusion should appear before the value. Comments are permitted.

```
1. Template inclusion
2. Value
```

A template inclusion is very simple:
`#"path/to/template"`

The path is relative to the location of the `.thars` file. If no extension is provided, `.thars.template` should be appended IF the target file does not exist.

Consider the following files:

```
- my_template.thars.template
- my_template
- my_other_template.thars.template
```

Assuming all are valid parseable template files, the following should be true:

```
// Points to "my_template.thars.template"
#"my_template.thars.template"

// Points to "my_template"
#"my_template"

// Points to `my_other_template.thars.template`
#"my_other_template"
```

For the value part of the file, it may either be a unit or a hierarchy.

An example of a valid `.thars` file:

```
// "templates/employee.template.thars"
#"templates/employee"

{
    "name": "Joe",
    "jobs": 3,
    "salary": USD(2_000e-3),
}
```

##### .thars.template

The `.thars.template` file exists in spite of a programmatic construction for some important reasons:
1. The developer may dynamically create templates.
2. The developer may store templates.
3. Users may specify which template they would like to use, if this is a desired behaviour.
4. Users may directly ascertain the full file requirements of their `.thars` file.
5. Users may modify templates, if this is a desired behaviour.
6. 3rd-party analysis tools may read it and provide real-time feedback on user THaRS.

It comes down to improved developer-user communication and more flexible application. Templates from files may be read and used directly, or may only exist as a front-end tool with an unmodifiable version existing on the back-end, all to the developer's specifications.

For a more strict but less documented, see the grammar file: [thars.template.ebnf](grammar/thars.template.ebnf)

Given a blank template file, several option are available out of the gate: one may declare [predicates](#predicates) [structs](#template-structs), [pseudotypes](#pseudotypes), or define the template. Structs and pseudotypes, as previously discussed, exist to serve the template. Structs and pseudotypes may be declared in whatever order, but only after the predicates are issued, and before the format is defined.

```
predicate "core"

// As awkward as all of this, it's actually allowed.
struct Foo {
    Baz
}
pseudo Bar = Foo
struct Baz {
    Bar
}

{
    "foo": Foo,
    "bar": Bar,
    "baz": Baz,
}

// This is where we get some problems though...
struct Qux {}
```

Recursive attributes are okay, but good luck inputting them in the `.thars` file.

The only problem with the previous example was that a struct declaration appeared after the template definition. Put simply, the template definition *must* be the last thing to appear in the file. This also implies there may not be two or more template definitions inside a template file, as this would make at least one of the template definitions *not* the last to appear in the file. A similar stance is taken when there are zero template definitions, as then a template definition will not be the last thing in the file.

```
1. Predicates (zero or more)
2. Structs/Pseudotypes (zero or more)
3. Template definition (one)
```

### Implementation
##### Grammar

The grammar is EBNF described in ISO/IEC 14977 (1996).
[thars.ebnf](grammar/thars.ebnf)
[thars.template.ebnf](grammar/thars.template.ebnf)

##### Faithfulness

'Faithfulness' is subjective, but an objective interpretation is attempted anyway in order to perceive the interoperability of THaRS between two languages. Thus faithfulness is a measure of how well an implementation implements all the features of THaRS described in this document.

Faithfulness is composed of several parts:
`[date] [value-score][grade]/[template-score][grade]`

`[date]` indicates the date of the commit of this document that the implementation adheres to, and that the faithfulness is based upon. It is based upon the gregorian calendar, and shall always be in the format `year.month.day` where `year`, `month`, and `day` are all integers.

`[grade]` denotes how the 'perfection' of a score in consideration of the `[date]`.
- `A` suggests the score is perfect.
- `B` suggests the score is imperfect.
- `C` suggests the score is minimal.

A perfect score is a score that is as high as can be. An imperfect score indicates the score ladder was skipped. A minimal score suggests the score ladder was not skipped, but is not perfect.

Scoring works like a ladder: in order to achieve one rung, all previous rungs must have been met. The score will thus denote the importance of features. Each rung is given a numeral value, and may have sub-rungs, all of which must be met in order to proceed to the next rung.

| feature                       | `value-score` |
|-------------------------------|---------------|
| Booleans                      | 0.1           |
| Rationals                     | 0.2           |
| ASCII Strings                 | 0.3           |
| Sequences                     | 0.4           |
| Maps                          | 0.5           |
| UTF-8 Strings                 | 0.6           |
| Integers                      | 1.0           |
| Template inclusion            | 2.0           |
| [Implicit extensions](#thars) | 2.1           |
| Structs                       | 3.0           |
| Full grammar adherence        | 4.0           |

| feature                                 | `template-score` |
|-----------------------------------------|------------------|
| `boolean` type                          | 0.1              |
| `rational` type                         | 0.2              |
| `string` type                           | 0.3              |
| Dynamic `[]` sequence type              | 0.4              |
| Dynamic `{}` map type                   | 0.5              |
| `integral` type                         | 0.6              |
| Static `[]` sequence type               | 0.7              |
| Static `{}` map type                    | 0.8              |
| Remainders                              | 0.9              |
| Default map values                      | 0.10             |
| `any` type                              | 0.11             |
| Qualifiers                              | 1.0              |
| Default map values subject to qualifier | 1.1              |
| Type ticket                             | 1.2              |
| RegEx ticket                            | 1.3              |
| Exclusive range ticket                  | 1.4              |
| Inclusive range ticket                  | 1.5              |
| Collapsers                              | 1.6              |
| Structs                                 | 2.0              |
| Predicates                              | 2.1              |
| Pseudotypes                             | 2.2              |
| Rawtext construction                    | 3.0              |
| [Implicit extensions](#tharstemplate)   | 3.1              |
| Full grammar adherence                  | 3.2              |

`2025.05.13 1.0B/0.6C` may be an example of an implementation that includes all value-score features up to and including 1.0, all template-score features up to and including 0.6. But the B-grade would also suggest that one or more value-score features exclusively greater than 2.0 was implemented. The C-grade would indicate that no template-score features beyond 0.6 were implemented.

When comparing implementations, one could easily tell the compatibility across languages by comparing the faithfulness. For example, another implementation with a faithfulness of `2025.05.13 4.0A/2.2B` will tell you exactly what features may be commonly used: structs, for example, will be a no-go.

An implementation is 'faithful' if it is up-to-date, and has A-grade scores across the board.

### Safety
##### Structs {#safety-tructs}

Structures are reflected on the developer's end, which may have side effects. Although this is not really a viable method of trojan, as one should not be executing a program they do not trust to begin with, the user should be aware of the potential for harm through misuse of a program via structure declaration.

#### Templates {#safety-templates}

Using a template in a `.thars` file will send a request to the implementation to handle the template file. It is the program developer's responsibility to acknowledge the requested file and either permit or deny its usage, and a faithful implementation will easily allow control over this.