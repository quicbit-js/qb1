# qb1 Encoding Foundations

## JSON Types and Constants 

qb1 directly supports the 3 JSON tokens and 5 JSON types

    name                    description
    
    null                    same token as in JSON, usable anywhere a value can be used
    true                    same token as in JSON
    false                   same token as in JSON
    
    decimal ("number")      an unlimited integer or decimal value known as "number" in JSON
    string                  string of characters encoded as UTF-8
    object                  a dictionary or map with string keys and any values
    array                   ordered list of any values
    boolean                 true or false

    
JSON number imposes no precision or length limits and 
JSON string, object, and array impose no member or length limits.  This is one of the
aspects that makes JSON so broadly adaptable to different data needs.

## Additional Number Types

The JSON number type is very broad, representing signed and unsigned integers as well
as decimal floating point.  To help bridge a variety systems and system requirements to data, 
qb1 supports the following additional subtypes:

    name                    description

    float                   base 2 floating point number
    rational                unlimited precision rational (p/q) capable of losslessly representing any numeric value
    integer                 integer is just a subset of the other three number types, but is a convenient way to declare a non-fraction value
    number                  an way to declare a field to be any qb1 number type (decimal, float, rational, integer)

Just like JSON's Number, qb1 number types have unlimited precision and/or size.  
While decimal can represent any integer or decimal fraction and is a good choice for accounting, but
can cause headaches when working with base-2 floating point which is javascript's default.  To 
losselessly represent base-2 values, qb1 supports the 'float' type.  The integer type is 
a subset of both decimal and float.  Rational is a superset of all these types as it can
represent any numeric value.  While different languages and applications can have efficiency
constraints that require using approximations of these values, it is important for 
a universal format to not impose any such constraints which is why *qb1 stores and loads all numeric types
quite efficientcly*.

**Choosing a number type** 

When working with javascript, you may find that float gives the both the best performance and
accuracy for large amounts of processing, while a decimal type may be perfect for holding 
currency values for certain fields.  Rational may be slow to work with on your system, but as computers
advance, may become the best choice.  Using 'number'

## Binary Types

qb1 adds two types to facilitate working with binary

    name                    description
                            
    byte                    alias for unt8
    blob                    array of bytes
    
qb1 blobs are representable in JSON using integer arrays or, more compact base64 strings and 
can be represented with or without indicator codes.

## Any

qb1 supports compound type expressions.  The 'any' type is shorthand for the
compound expression for all types:

    name                    description
    
    any                     same as  null|boolean|string|number|blob|[*]|{*:*}
                                 or  nul|b|str|num|blb|[*]|{*:*}
                                 or  N|B|s|n|x|[*]|{*:*}
                                 or  *

See [compound types with type lists](compound-types-with-type-lists) and 
[short names and codes](short-names-and-codes) for more.

### Numeric Size Constraints

For every number subtype, qb1 defines common size-contrained types.  
(here we use some  [short type names and codes](#short-names-and-codes).  int32 == integer32.  They are 
equivalent)

    name                            description
                                    
    int8                            8 bit signed integer
    unt8                            8 bit unsigned integer
    flt32                           32 bit float
    f32                             32 bit float (short-code)
    dec64                           64 bit decimal
    d64                             64 bit decimal (short-code)
    rational64                      64 bit rational 

The pre-defined number sizes are limited to the common sizes 8, 16, 32, 64, 128 for integer and
uinteger and 32, 64, and 128 for float and decimal and 64 or 128 for rational.

More examples are given in [the full list of size-constrained numbers](#size-constrained-numbers) below.

### String and Blob Size Constraints

For placing or communicating byte limits on strings and blobs, qb1 allows byte
specifiers:
 
    string<n>           str<n>              s<n>            string with limit of n characters
    blob<n>             blb<n>              x<n>            array of n bytes
    
For example
    
    string32  - a string of 32 bytes or less
    blob1024 -   blob of 1024 bytes or less


## Compound Types with Type Lists

Specifying a compound type using list of possible types can be done 
with parenthesis and the pipe operator for logical-or:

    expression                      description
    
    (string|null)                   a string or null
    (int32|flt32)                   a 32 bit float or decimal
    (i32,f32,d32)                   a 32 bit integer, float, or decimal *
    any                             short-hand for any type: (null|true|false|array|object|string|number|blob)
    
\* [using short codes](short-names-and-codes)
    
## Nested Types

Objects and arrays can be expressed using the same tokens used in JSON:

    expression                      description
    
    [ string ]                      an array of strings
    { string: int32 }               an object with string keys and 32-bit integer values
    { string: (string|null) }       an object with string keys and 32-bit integer values
    { string: (string256) }         an object with string keys and string (max 256 byte) values
    
(whitespace is optional)

## Short Names and Codes

qb1 supports 3-character short names and single character codes for JSON types and
symbols:  
  
    long name           short name          code
    
    string              str                 s
    object              obj                 o
    array               arr                 a
    number              num                 n
    true                tru                 T
    false               fal                 F

qb1 supports 3-character short names and single character codes for other basic types:

    long name           short name          code
    
    integer             int                 i
    decimal             dec                 d
    uinteger            unt                 u
    float               flt                 f
    rational            rat                 r
    boolean             boo                 b      
    byte                byt                 X
    blob                blb                 B
    
    (blob and byte codes are capitalized to distinguish from other types)

### Size-Constrained Numbers

For interaction with common languages and architectures, qb1 defines the following common 
size-constrained numeric types:

    long name           short name          code-form       description
    
    integer8            int8                i8              8 bit signed integer
    integer16           int16               i16             16 bit signed integer
    integer32           int32               i32             32 bit signed integer
    integer64           int64               i64             64 bit signed integer
    integer128          int128              i128            128 bit signed integer
    uinteger8           uin8                u8              8 bit unsigned integer
    uinteger16          uin16               u16             16 bit unsigned integer
    uinteger32          uin32               u32             32 bit unsigned integer
    uinteger64          uin64               u64             64 bit unsigned integer
    uinteger128         uin128              u128            128 bit unsigned integer
    float32             flt32               f32             IEEE 754 single precision float
    float64             flt64               f64             IEEE 754 double precision float
    float128            flt128              f128            IEEE 754 quadrupal precision float
    decimal32           dec32               d32             IEEE 754 decimal32
    decimal64           dec64               d64             IEEE 754 decimal64
    decimal128          dec128              d128            IEEE 754 decimal128
    rational64          rat64               r64             rational stored as 2 unt32 values
    rational128         rat128              r128            rational stored as 2 unt64 values
    
### Compatibility with JSON

For the JSON types, 'qb1' format can write equivalent JSON and binary
formats.  That is, the information that it stores in either
form will generate the exact same decoding result.  This includes representation of any-magnitude integer,
any-precision of base 10 decimal value and any length text or blob.

We refer to both binary and json encodings as 'qb1' format, because they are both lossless supported
encodings.  When we want
to specify the type of encoding, we use hyphenated terms "qb1-json" or "qb1-binary".

For values that are outside of the JSON type set, such as raw binary and base 2 floating point, qb1 provides
lossless translation as well as rounding options.

 Full Representation

    qb1-binary           qb1-json                         qb1-binary

    qb1 base-2  ------>  JSON (hex string)  ----------->   qb1 base-2
    float                 "0xF312.835AC"                    float

    qb1         ------>  JSON (rational string) ------->   qb1
    rational              "355/113"                         rational

    qb1         ------>  JSON (base 64 string)  ------->   qb1
    raw binary            "3E12F07C1AC367002A"              raw binary

We refer to these JSON representations of values as "qb1-json"

Rounding (when size-savings are more important than exact binary match):

    Machine      ------>  JSON (decimal)      ----------->  Machine
    float64               rounded                          float64
    approximately 0.1     0.1                              approximately 0.1 (not exactly same as original)

### Compatibility with JSON as Self-Described Types

JSON encodes type information within value format.   This JSON snippet:

    { "stat": [ 1.25, 1.34 ], "live": true, "type": "skewed", "props": null }

Demonstrates values of each of the six JSON types: object (the entire object), array, number, boolean, string and null.

#### Effectiveness of inter-compatible text and binary

The inter-operability that qb1 offers between JSON and binary is a special foundation for working with data:

1. qb1 embeds binary data within JSON documents for faster read performance or other needs.
1. qb1 embeds JSON data within binary for higher transparency and compatibility with systems.
1. qb1 can create 100% pure binary or pure JSON versions of any qb1 information desired.

These properties allow us to work with binary in whatever proportion is required and leave other data
human-readable, or to pass binary payloads through systems that handle JSON documents, or to debug binary
formats by looking at their form in JSON.  The ability to choose at any level enables new capabilities and
improved productivity.

#### Extended Capabilities

Inter-operability of JSON/binary can be a nice tool, but it becomes truly transforming when we leverage
some of other qb1 offerings.

* qb1-schema
* qb1-ref
* qb1-pack
* qb1-scan

Perhaps the most important aspect of the qb1 encoding scheme is that *it is designed to enable further optimizatios*.

create new binary encoding strategies on top of qb1micro.  Unlike traditional binary forms, encoding definitions
defined in qb1 as well as the values themselves
can be shared in full fidelity using plain JSON.  This means that specifications that describe these binary encodings can embed actual
examples of encodings and values in human-readable form.  Issues with binary data can be documented and resolved just as one would
with plain text by looking at the JSON equivalent form.

## Language-Oriented Types

qb1 APIs provide comprehensive type mappings in different languages

### C to qb1
<pre>
  C type             qb1 type     comment
                     
  NULL               nul
  bool               boo
  char16_t           unt16
  char32_t           unt32
  int8_t             int8
  int16_t            int16
  int32_t            int32
  int64_t            int64
  uint8_t            unt8
  uint16_t           unt16
  uint32_t           unt32
  uint64_t           unt64
  float              flt32
  double             flt64
  str::string        str          or blob to preserve encoding other than utf-8
  str::wstring       str          or blob to preserve wide encoding
  str::u16string     str          or blob to preserve utf-16 encoding
  str::u32string     str          or blob to preserve utf-32 encoding
  []                 arr 
  map_t*             obj          a map maintaining order of added entries
  char*              blb
</pre>



### Java to qb1
<pre>
 Java type                      qb1 type       comment
                                
  null                          null
  boolean                       boolean
  char                          unt16
  byte                          int8           Java bytes are signed
  short                         int16
  int                           int32
  long                          int64
  float                         float32
  double                        float64
  java.math.BigDecimal          decimal
  []                            array
  java.util.List                array
  java.util.LinkedHashMap*      object
  java.lang.String              string
  java.nio.ByteBuffer           blob
</pre>

\*maintaining order of map entries is an important aspect for serialization and checksum.

## qb1's Approach to Types

To understand the type system and options available, it is helpful to call out the several facets and meanings of
of this "type" concept.  Types are used to *constrain*, *summarize*, *encode*, and *semantically describe* values.

1. constrain

   A type can be regarded as allowing a subset of possibile values.  In this way of thinking, blob
   doesn't constrain.  string constrains a value slightly (to legal character values).  number constrains values
   to valid numbers*, integer to the subset of integer numbers, uinteger to unsigned integers, etc.  Custom
   types can constrain values even further.  A "zip-code" type could constrain to only valid zip code values, and
   a "color" type could constrain to a list of known color names, for example.

   \* though series of numbers can represent any value, which muddies the waters a bit when we think of constraining.

1. summarize

   Very similar to constraining, a type summarizes value sets along defined boundaries.  The value set [1.2, 1, 3.4] could
   be summarized as an array of numbers: ["num"], while [-1, 3, 4] could be more specifically summarized as an
   array of integers: ["int"] and [1, 3, 4] even more specifically as unsigned integers: ["uint"], or more
   specifically to ["int(1:4)"], integers ranging from 1:4.  Notice using "uint" instead of "int" is acceptable, but
   not required when ranges of unsigned integer are given.

1. encode

   A type describes to qb1 how values are to be encoded/decoded to and from bytes.  But note that encoding
   and decoding is a multi-layer task.  For example, qb1's rational, float,
   and decimal values are all read and written using the same form: a sign flag with two unsigned integers,
   while functions a layer above perform more encoding operations to and from the values:
   floats and decimals are composed into IEEE format, while rationals are placed into language-specific objects that
   hold onto the underlying separate values.

1. semantically describe

   Type semantics can be tricky to pin down to specifics or universal guidelines, but suffice to say that
   an important application of types is to narrow down the semantic scope of values.  'number' has broad semantic
   meaning, 'unsigned integer' narrows down further.  'person age' is narrower still. 'patient age'
   then narrows down the meaning into a specific semantic in a business domain.
   Objects (or data records) hold more domain and application specifics with their implied or explicit associations between
   types within an object.  As a general rule, stand-alone type definitions such as 'person age' have greater stability and system cross-compatibility
   than full record definitions that associate types together such as
   {"patient":{"name":"string", "insurance":"string", "age":"person-age", ...} }.  A good guideline for modelling domain types is
   to **be cognizant of the scope of data sets that underly a type definition**:  e.g. 'age' in this record
   means 'patient age', which is a subset of the concept 'person age'.  If we were building a palet of types
   to share across different forms and applications then 'person age' is probably
   a good choice for a stand-alone type.  Constraints that apply to 'person age' also apply to 'patient age', but not vice-versa.
   We could create a 'patient age' general type, but if those constraints are the same as person age, then 'patient age'
   adds unneeded constraints and unnecessary type complexity.

Programming to high-level type abstractions like 'number' can simplify programs, saving time, and reducing errors,
but often with a performance cost.  JSON lies on the convenient side of this spectrum while C aligns with performance
and Java sits somewhere between these two.  Javascript started out with a strong bias for convenience but has made
a move towards higher performance with the introduction of typed arrays and faster interpreters, opening
new server/browser possibilities.

qb1 APIs and types allow developers the convenience of working at the JSON abstraction level, and also,
to use more specific and higher performing encodings and constraints, at their option.  This is achieved
by creating binary-to-binary conversions that run faster than JSON-to-binary and by reducing the
number of conversions made in many situations.

Transparency
------------
Unlike most binary encodings, qb1's encoded values translate directly to JSON, making files
intuitive and accessible\*.  qb1 provides a number of useful command-line tools that can select and summarize large
data sets, or expose information as JSON, CSV, or other format.

qb1 can also create lossless hybrid files with rich human-readable data summaries
preceding compact binary data encodings, which are operationally more convenient than pure
binary formats because contents can understood with any text viewer.

\*when accessibility is *not* wanted, qb1 gives clients other options.  See 'Security' section for how qb1 can protect data.

## Security

qb1 transparency is an important factor for improving access and productivity of developers and operations personnel
working with data.  But qb1 transparency is optional.  qb1 data files can secure any or all data contents using
SHA-3 or other plug-in encryption function.  Encryption can be aimed at the file level, column level, and/or
summary/meta level.

## Consistency

qb1 provides high levels of structured consistency checking and recovery mechanisms for ensuring accurate
file contents.

## Custom Types

qb1 allows clients to define simple types through references to other types with added constraints.
Using qb1schema, clients can define and compose complex types from other types.
Using qb1-ref, clients can cleanly manage schema versions and complex schema compositions.

qb1 APIs support three mechanisms for extending the type set:

1. Static

   Load and cache types from a file or service when the API is initialized

2. Dynamic

   Runtime definition of types using API calls.  This can be disabled.

3. Reference

   Look up types according to references and domain maps that configure namespaces either locally,
   within an internal network, or chosen public servers - or using some combination of the three.
   References can be static or floating.  Floating references are, by definition, automatically
   updated and qb1 ensures that schemas containing floating (unspecific-versions) are themselves
   floating (transitive property of composition).

The type registry mechanism makes use of qb1-ref, qb1's universal
referencing strategy over well-defined namespaces for shared types.


# The Foundation of qb1 Encoding

In an attempting to make the foundations of qb1 simple and flexible, all qb1 encodings are built upon
a 2 "particles" comprising just 4 "atoms".  Through this foundation, qb1 achieves elasticity
and precision and becomes parsable with selective degrees of processing.

## Particles and Limitless Precision

At the lowest level, underpinning all of qb1's 7 core value types, and by extension all other qb1 types, are 2 particle
encodings.

    chain      a series of bytes with 7-bit values where the first bit is used to 
               define the chain length.  For example:
                
               0xxxxxxx                      (a single byte chain representing from 0x00 through 0x7F)
               1xxxxxxx 1xxxxxxx 0xxxxxxx    (a three byte chain representing 0x100000 through 0x1FFFFF)
    
    raw        One or more raw bytes (following a chain particle that encodes length):
                
               chain particle (one or more 7-bit encoded bytes)
               |
               |              raw particle of 2 bytes
               |              |
               length 2       byte 0xA3   byte 0xCF
               
               00000010       10100011    11001111   
               

Note that since both of these particles are of variable and unlimited length, and since all 
qb1 values are composed of these particles, therefore all qb1 values are of unlimited length and/or
precision.  The foundation is future-proof and can grow to any amount of physical storage and 
processing available.
This means all qb1 values can have any precision floating point, any size
integer, any magnitude exponent, any length string or blob.  These all can be as much as a system has space and time to process.

To review - an unsigned integer can be written as any number of chained bytes,
costing 1 bit for every 8 and so is applied by qb1 to value types that average under 8 bytes
in length.  Value types that average more than 8 bytes in length use the raw encoding which costs one extra byte
for encodings 1 through 127 bytes long, 2 bytes for encodings 128 through 32767, 3 bytes for encodings
2^(8 * 2 - 1) through 2^(8 * 3 - 1) and so makes sense for large byte sets such as long strings.

As a side note, the usage and importance of uint increases dramatically as we start to compact data
with run-length encoding strategies.


## Atoms and Self-Described Values

A self-described value consists of a "token" byte that describes the type of data and zero, one or
two subsequent particles that hold the value.  There are only 4 value configurations, 
we call "atoms" used to describe any value. (Minimizing the underlying structure to just four configurations means that
a basic decoder can parse through a file by implementing handling for only two particles 
in the 4 possible arrangements.)

When an atom follows a type-code, it is a "self-describing" value.  A token particle that 
needs no subsequent particles is a special case of self-describing value/token.  null, true,
false, and infinity are all examples of these self-describing value/tokens.  
Examples of single uint atoms include int, uint, and NaN errors.
Examples of double uint atoms include floating point and complex numbers.
Examples of raw atoms are blob and string.

Note that the typecode that works in conjunction with the codes below is only written preceding zero-to-two subsequent
bytes in
the qb1 simple form.  Higher performing qb1 formats
such as qb1pack will usually optimize by writing typecodes once for repeating same-type atoms.

<pre>
   Described values are written as a uint type-code followed by subsequent particles.

   nick-name       type-code              atom
                    |                    |
                    |            ----------------
                    |            |              |
                    |        particle 1     particle 2
   stand-alone     uint
   single          uint          uint
   double          uint          uint          uint
   raw             uint          raw

 Type Codes

 A type code is a uint particle, where the first bit is used to extend the code to any length:


 byte-1                       byte-2  (if extended)             byte-3 (if extended)         ...

 extension-bit                extension-bit
   |                            |
   |   atom    detail           |          detail
   |    |        |              |            |
   |  ----  -------------       |  ----------------------
   |  |  |  |           |       |  |                    |
   x  x  x  x  x  x  x  x       x  x  x  x  x  x  x  x  x

 This means that although all common type codes are a one byte, type type-code is decoded 
 using the same 7 bit integer chaining used for integers and types
 can be extended in the future to add any number of types.

 The following single byte type codes are defined

 stand-alone codes (having no subsequent values)

    hex    extend   atom            detail
   range     |       |                |
     |       |    ------    ---------------------
     |       |    |    |    |                   |
   00        0    0    0    0    0    0    0    0      null          (not preserved in objects by default - will be interpreted as not existing)
   01        0    0    0    0    0    0    0    1      explicit null (sticky in objects - will be preserved)
   02-03     0    0    0    0    0    0    1    T      boolean (T: 0=false, 1=true)
   04-05     0    0    0    0    0    1    0    S      infinity (S: sign 0=positive, 1=negative)
   06        0    0    0    0    0    1    1    0      NaN (numeric error, without error bit detail)
   08        0    0    0    0    1    0    0    0      object start
   09        0    0    0    0    1    0    0    1      object end
   0A        0    0    0    0    1    0    1    0      array start
   0B        0    0    0    0    1    0    1    1      array end
   0C        0    0    0    0    1    1    0    0      incremental mode
   0D-1F     (unused)

 codes followed by a single uint

    hex    extend   atom            detail
   range     |       |                |
     |       |    ------    ---------------------
     |       |    |    |    |                   |
   20-21     0    0    1    0    0    0    0    S      integer (S: sign 0=positive, 1=negative)
   22        0    0    1    0    0    0    1    0      NaN (numeric error, folloed by error code as bit-rotated* uint)
   23-3F     (unused)

 codes followed by two uint

  numeric  extend   atom            detail
   range     |       |                |
     |       |    ------    ---------------------
     |       |    |    |    |                   |
   40-45     0    1    0    0    0    R    R    S      2-part number (R: float/decimal/rational values, S: sign 0=positive, 1=negative)
   46-4F     (unused)

 examples
   40        0    1    0    0    0    0    0    0      positive binary floating point
   41        0    1    0    0    0    0    0    1      negative binary floating point
   42        0    1    0    0    0    0    1    0      positive decimal floating point
   43        0    1    0    0    0    0    1    1      negative decimal floating point
   44        0    1    0    0    0    1    0    0      positive rational (p/q)
   45        0    1    0    0    0    1    0    1      negative rational (p/q)

 codes followed by uint (length) and that number of raw bytes

  numeric  extend   atom            detail
   range     |       |                |
     |       |    ------    ---------------------
     |       |    |    |    |                   |
   60-61     0    1    1    0    0    0    0    U      blob (U: 0=raw bytes, 1=utf-8 encoded string)
   62-7F    (unused)


</pre>
## bit rotation (for floating point significand and error codes)

qb1 encodes signifand as a uint.  Here's how:

IEEE float significands are encoded MSB to LSB (here left-to-right) where zeros on the right are insignificant.

01010000 00000000 00000000    can be represented as 0101

Could we represent this as an integer, value 5, which is a single byte\*?

101

Not quite.  We lost the leading zero.  But we can rotate the right-most set bit to the left:

010(1)  ---->  (1)010

This retains all the significand information as a integer (10) and does fit in a single byte.  Reversing the process
restores the original value.


## uint 7-bit chaining

Chained unsigned integers use 7 bits of every byte for the integer content and the most significant bit to indicate additional bytes:

<pre>
00000000                           0
00000001                           1
01111111                         127
10000001 00000000                256
10000001 00000000 0000001   2^15 + 1
</pre>


Chained integers can be any-size integer
In practice, efficient handling is usually limited to
-(2^63-1) and 2^63 for signed integers and between 0 and 2^64 for unsigned.  Current integer limits in javascript are
between -(2^53-1) and 2^53-1).


<pre>
self-described-value :=
 type-code  |
 value      |

value :=
  self-described-value |
  value
</pre>

**type-code** describes the subsequent encoded value.

**value** is encoded using a combination of uint and raw bytes indicated by type-code.
