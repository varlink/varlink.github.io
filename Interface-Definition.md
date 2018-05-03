# Interface Definition

Interface files are named after the interface they implement with the suffix `.varlink`.

## Keywords

|Keyword          |Description                                                     |
|-----------------|----------------------------------------------------------------|
|interface        |The name of the interface in reverse-domain name notation. It must be the first keyword in the description.|
|[type](#type)    |A custom type. The definition of an object/record/structure.    |
|[method](#method)|A method with input and output parameters/objects               |
|[error](#error)  |The name of the error and and optional data describing the error|

## Type
With ```type``` a structure of fields (object) can be defined. Following the ```type``` keyword the type name has to be specified. Type names have to start with an uppercase character and continue with alphanumeric letters. Enclosed in ```()``` the declaration of the various fields follows. Field names start with a lowercase character and continue with alphanumeric letters. ```_``` can appear in the middle of the field name, but not at the beginning or at the end. Object declarations specify a field name followed by a colon and a type. Fields are separated by comma.

```go
type MyType (
   example_bool: bool,
   example_int: int,
   example_float: float,
   example_string: string,
   example_object: object,
   example_enum: (one, two, three),
   example_struct: (first: int, second: string),
   example_array: []string,
   example_dictionary: [string]string,
   example_stringset: [string](),
   example_nullable: ?string,
   example_nullable_array_struct: ?[](first: int, second: string),
   example_other_type: MyOtherType
)
```

|Type             |Keyword |Description                                    |Example                           |
|-----------------|--------|-----------------------------------------------|----------------------------------|
|Boolean          |bool    |_true_ or _false_                              |```flag: bool```                  |
|Integer          |int     |Implementation-specific size, usually `int64`  |```whole: int```                  |
|Floating Point   |float   |Implementation-specific size, usually `IEEE754`|```half: float```                 |
|String           |string  |                                               |```name: string```                |
|Object           |()      |Structure of named types, separated by comma   |```(first: int, second: string)```|
|Enum             |()      |List of names without types, separated by comma|```(this, that, other)```         |
|Map/Dictionary   |[string]|Map of types with a unique string as key       |```[string]int```                 |
|Array            |[]      |Array of types                                 |```[]int```                       |
|Nullable         |?       |Nullable type/Maybe                            |```?string```                     |
|Foreign object   |object  |Foreign untyped/raw object                     |```payload: object```             |

## Method
Method names start with an uppercase character and continue with alphanumeric letters and specify an input and output parameter object.
```go
method Add(a: int, b: int) -> (sum: int)
method Foo(a: int, b: MyType) -> (bar: MyOtherType, baz: float, more: (i: int, f: float, s: string))
```

## Error
Error names start with an uppercase character and continue with alphanumeric letters and optionally carry an error object describing the error. The error is identified by its fully-qualified name. Varlink calls can only return well-defined errors specified in the interface description.
```go
error UnknownAction(action: string, more_data: DetailedError)
```

## Grammar
All whitespace (space, `\t`, `\r`, or `\n`) is ignored, except inside comments, which are started with `#` and extend to the next `\n`.

Comments that start on a new line and are immediately followed by `interface`, `method`, `type` are documentation for that following declaration. These comments may span multiple lines.

```
interface
        "interface" interface_name members

members
        member
        member members

member
        type_alias
        method
        error

type_alias
        "type" name object
        "type" name enum

method
        "method" name object "->" object

error
        "error" name object

type
        element_type
        "[]" type
        "[string]" type
        "?" element_type
        "?" "[]" type
        "?" "[string]" type

element_type
        struct
        enum
        name
        "bool"
        "int"
        "float"
        "string"
        "object"

struct
        "(" struct_fields ")"

struct_fields
        struct_field
        struct_field "," struct_fields

struct_field
        field_name ":" type

enum
        "(" enum_fields ")"

enum_fields
        field_name
        field_name "," enum_fields

interface_name
        a valid internet domain name with its components reversed

name
        [A-Z][A-Za-z0-9]*

field_name
        [A-Za-z](_?[A-Za-z0-9])*
```

## Parsing Expression Grammar
```peg
whitespace /* Modeled after ECMA-262, 5th ed., 7.2. \v\f removed */
  = [ \t\u{00A0}\u{FEFF}\u{1680}\u{180E}\u{2000}-\u{200A}\u{202F}\u{205F}\u{3000}]

eol_r /* Modeled after ECMA-262, 5th ed., 7.3. */
  = "\n"
  / "\r\n"
  / "\r"
  / "\u{2028}"
  / "\u{2029}"

comment
    = "#" [^\n\r\u{2028}\u{2029}]* eol_r

eol
    = whitespace* eol_r
    / comment

_
    = whitespace / comment / eol_r

field_name
    = [a-z](_?[a-z0-9])*

name
    = [A-Z][A-Za-z0-9]*

interface_name /* no hyphen at begin and end */
	= [a-z]+ ( '.' [a-z0-9]+ ([-] [a-z0-9]+)* )+
	  / "xn--" [a-z0-9]+ ( '.' [a-z0-9]+ ([-] [a-z0-9]+)* )+
dict
    = "[string]"

array
    = "[]"

maybe
   = "?"

element_type
    = "bool"
    / "int"
    / "float"
    / "string"
    / "object"
    / name
    / venum
    / vstruct

type
    = element_type
    / maybe element_type
    / array type
    / dict type
    / maybe array type
    / maybe dict type

venum
    = '(' ( field_name ** ',' ) _* ')'

argument
    = _* field_name _* ':' _* ( type ++ ',' )

vstruct
    = '(' ( argument ** ',' ) _* ')'

vtypedef
    = "type" _+ name _* vstruct
    / "type" _+ name _* venum

error
    = "error" _+ name _* vstruct

method
    = "method" _+ name _* vstruct _* "->" _* vstruct

member
    = _* m:method
    / _* t:vtypedef
    / _* e:error

interface
	= _* "interface" _+ interface_name eol ( member ++ eol ) _*
```
