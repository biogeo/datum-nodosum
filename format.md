# A format for storing data in a human readable/writable fashion

## Terminology

* **document**: a stream of text describing a set of data, which includes markup to indicate the structure of the data.
* **markup**: symbols in an particular context which provide structure to the document.
* **presentation**: the way a section of the data appears literally in the document.
* **representation**: the data structure encoded by the document.
* **interpretation**: the mapping from presentation to representation.
* **generic representation**: a tree-like data structure bearing the abstract structure of the data in the document, but lacking a complete interpretation.
* **line flow** or **flow**: the interpretation given to newlines and indentation. Several flow modes exist depending on the current context.
* **element**: a unit of data in the document.
* **atom**: an element which, for the purposes of representation in the document, has no substructure.
* **collection**: an element with substructure.
* **type**: the interpretation of a particular element in the document to specify its representation from its generic representation.
* **implicit type**: the default type assigned to an element based on its content, structure, and/or presentation.
* **explicit type**: a type assigned to an element with a type tag.
* **type tag**: an identifier preceding an element to specify its type, taking the form `type::`.
* **blank line**: a line containing only whitespace.

## Generic representation

The first step in interpreting a document is to construct its *generic representation*, parsing its markup to identify the elements and their structure. Each element in the document is either a **generic atom**, or a **generic node**. A generic atom is a string containing the atom's presentation (after interpreting flow) together with an implicit or explicit type. A generic node consists of **parent element** together with zero or more ordered **child elements**. If all parent elements are atoms, then the data structure is simply a tree, but node parents are permissible.

## Indentation

Naively, **indentation** is the number of spaces at the start of a line, but other factors may also affect indentation. In general, consecutive lines with the same indentation are semantically at the same "level" in the document. The **indentation level** of a line is the number of "steps" in indentation to get to a particular line's indentation. No indentation is indentation level 0. The first line with indentation sets indentation level 1. The next line with greater indentation sets indentation level 2, and so on.

When parsing a line, the presentation undergoes **trimming** to remove whitespace: the indentation is removed, as is any trailing whitespace before the newline. Trimming happens after processing the delimiter but before processing the line wrap.

## Delimiter mode

There are three delimiter modes which control the way newlines are used as delimiters between elements. In all modes, literal delimiters provided by markup will delimit elements. Note that multiple sequential delimiters in most contexts are "squashed" into one; a null/missing value must be specified explicitly.

* **Line delimiting**: Each newline is a delimiter.
* **Block delimiting**: Each blank line is a delimiter. A newline followed by a change in indentation is also a delimiter.
* **Literal delimiting**: Newlines have no delimiting function; only literal markup delimiters apply.

## Line wrap mode

There are three line wrap modes which are used in various contexts. The aim of the line wrap rules is to be transparent to a human reading a document, minimally burdensome to a human writing a document, and unambiguous to an algorithm parsing a document. That is, for a human reading a document, the line wrap rules should be "obvious" based on the structure of the data in the text. A human writing a document may need to be aware of the line wrap rules, but which rule applies in a particular context should be straightforward.

The line wrap modes are:

* **Paragraph wrapping**: Individual newlines are lexically replaced with a single space. Blank lines are lexically replaced with newlines.
* **Block wrapping**: Individual newlines are lexically discarded. Blank lines are lexically replaced with newlines.
* **Non-wrapping**: All newlines are lexically preserved as presented.

## Quoting

There are several quoting methods which force the quoted content to be treated as an atom and markup to be ignored (other than escape sequences). For the purposes of stripping, every line in a quoted atom is indented to the level of the least-indented line.

* `'Single'` or `"double"` quotes, uses *non-wrapping* mode. Initial and final newlines are ignored, and stripping is performed.
* `'''Triple single'''` or `"""triple double"""` quotes, uses *paragraph wrapping*. Initial and final newlines are ignored, and stripping is performed.
* Beginning with a line `--- BEGIN X ---` and ending with `--- END X ---` (where `X` may be any text but must be the same for both), uses *non-wrapping* mode, and escape sequences are not interpreted. Stripping is performed.
* Beginning with a line `+++ BEGIN X +++` and ending with `+++ END X +++`, uses *block wrapping* mode, and escape sequences are not interpreted. Stripping is performed.
* Beginning with a line `~~~ BEGIN X ~~~` and ending with `~~~ END X ~~~`, uses *non-wrapping* mode, escape sequences are not interpreted, and stripping is not performed.

## Atoms

### `null`

Representation of a missing or empty value. The canonical presentation is an empty string; in places where a token is required, `null::` may be used.

### `number` (or `real` or `real number`?)

Representation of a numeric value. The canonical presentation is a decimal value. Implicitly, no distinction is made between various numeric types and an application is free to choose a representation most suitable for the data, but an explicit type may be provided with a type tag. The default numeric subtypes are:

* `integer`: a signed integer value, canonically presented as an optional sign (`+`/`-`) followed by one or more decimal digits, as in `-12`.
* `unsigned` or `natural`: an unsigned integer value (natural number), canonically presented as one or more decimal digits, as in `42`.
* `float`: a floating point (or decimal; implementation is not important) value, canonically presented as an optional sign followed by a sequence of digits and exactly one decimal point, then optionally an `e` or `E` and an integer, as in `3.1415` or `6.022e23`.
* `rational`: A rational number, canonically presented as a fraction of integers, that is, an integer, zero or more whitespace, a `/`, and a natural number, as in `-3 / 13`.

An atom whose presentation matches any of the above is implicitly interpreted as a `number`.

### `string`

Representation of text. An atom whose presentation does not match any other implicit type rule is implicitly interpreted as a `string`. Additionally, any quoted atom is implicitly interpreted as a `string`.

## Collections

### Associative pair

### List

### Set

### Tree node

### Associative map

## Atomic types

### `string`

The most primitive atomic type is `string`; no particular structure is required
in the expression of the atom. Bare text that fails to meet the requirements for
any other atomic type is implicitly interpreted as a `string`.

#### Example
```
This is a string.
```

parses as itself.

Single newlines in a bare string are converted to a single space; double newlines
terminate the atom.

#### Example
```
This is also
a string.
```

parses as
```
This is also a string.
```

Additionally, an atom beginning and ending with `"` or `'`




---------

This is the first item in the document
    This is a child item because its indentation increased
This is the second item at the same level as the first
This is still part of the second item because of paragraph wrapping

This is the third item at the same level as the first
* This is the first item in a list which is the fourth item at the top level
* This is the second item in that list
  This is still part of the second item because of paragraph delimiting
  Also note that the bullet increased the indentation by 2
  * This is a child of the second list item

    How do we handle this case?
This is the fifth item at the top level: And this is its sole child
This is the sixth item because pairs force line delimiting
* A list with a key: value pair
  What about this key: value pair?
