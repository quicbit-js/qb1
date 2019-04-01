# Describing Complex and Composite Values

qb1 allows one to serialize and store every kind of complex *value*.  And every value can be described by a
*type*.  The qb1 *types* themselves are *values* and are described by types.  At
the base of the definitions sit the "base" types.  The base types are the 
foundation that describe all possible types and values.

Every qb1 value can be self-described using the form
    
    { 
        $type: ...,
        $value: ...
    }

A self-described value can be assigned a name and description as well:

    {
        $type: ...,
        $name: ...,
        $description: ...,
        $value: ...
    }
    
Type definitions in qb1 are values as well and can be named and described.  

    {
        $type: 'type',
        $name: 'patient_age',
        $description: 'age of a patient',
        $value: 'int'
    }
    
Types can be simple or complex as explained in the following section.

## Type Overview

As described above, qb1 values all have a type.  All qb1 types are values with the type 'type'.

    foundation          type        $type: 'type'           (type is itself)
    
In addition to a type, all qb1 types have a $base type.  There are 16 base types.  These types
all have themselves as their base:
    
qb1 base types: 

    base types          type            $type: 'type'       $base: 'type'
                        integer         $type: 'type'       $base: 'integer'
                        boolean         $type: 'type'       $base: 'boolean'
                        string          $type: 'type'       $base: 'string'
                        number          $type: 'type'       $base: 'number'
                        decimal         $type: 'type'       $base: 'decimal'
                        integer         $type: 'type'       $base: 'integer'
                        array           $type: 'type'       $base: 'array'
                        object          $type: 'type'       $base: 'object'
                        ...

When describing a type, qb1 accepts three physical forms:

    object              objects can be used to define any simple or complex type 
    array               a convenient form for defining simple or cycling array types (more concise than object form)
    string              a type expression which may be just a name such as 'integer', a path 'mytypes/integer', 
                        or an expression 'integer|null'.

Using the 16 base types as a foundation, users can construct custom types of any structure or form using 
qb1's intuitive syntax:

    {
        $type: [
            {
                id: 'integer',
                tags: [ 'string' ],
                address: { 
                    street: 'string',
                    city: 'string',
                    state: 'string',
                    zip: 'integer'
                }
            }
        ],
        $value: [ 
            { 
                id: 32,
                tags: [ 'red', 'green', 'blue' ],
                address: {
                    street: '42 Main Street',
                    city: 'Portland',
                    state: 'Maine',
                    zip: 07423
                }
            },
            { 
                id: 45,
                tags: [ 'yellow', 'purple' ],
                address: {
                    street: '77 Tuscon Rd',
                    city: 'Tuscon',
                    state: 'Arizona',
                    zip: 73255
                }
            },
            ...
        ]
    }

... or apply more constraints ($stipulations):

        $type: [ 
            {
                id: 'integer',
                tags: [ 'string' ],
                address: { 
                    street: 'string',
                    city: 'string',
                    state: 'string',
                    zip: 'integer'
                },
                $stip: {
                    'id': { lo: 0, hi: 999, require: true },
                    'address/zip': { lo: 0, hi 99999 }      
                }
            }
        ],
        $value: [ 
            { 
                id: 32,
                ...
                
... or apply some of the short-hand expressions such as '!' (required field) and '0..999' 
integer range to summarize constraints within the simple type structure:

        $type: [ 
            {
                'id!': '0..999',
                'tags': [ 'string' ],
                'address': { 
                    street: 'string',
                    city: 'string',
                    state: 'string',
                    zip: '0..99999'
                }
            }
        ],
        $value: [ 
            { 
                id: 32,
                ...

Custom types cannot be extended (cannot be base of other types), but qb1 has a number
of features for supporting type composition such as embedded types and references.

## A More Intuitive Type System - Comparison with Avro and JSON Schema

qb1 self-described values can be expressed as a self-describing stand-alone object using $type and 
$value properties.  Many of us don't think about it much because it has become intuitive for most
of us, but simple types in javascript and JSON are discerned from use of quotes, brackets and braces:

javascript: 

    'hi'                    - a string
    4                       - a number
    [ 1, 2, 3 ]             - an array of numbers
    { a: 1, b: 'hi' }       - an object with fields 'a' and 'b'
    
JSON:

    "hi"                    - a string
    4                       - a number
    [ 1, 2, 3 ]             - an array of numbers
    { "a": 1, "b": "hi" }   - an object with fields 'a' and 'b'


What these languages are missing a system that is *just as intuitive* as these structures that formally 
declares the type structures themselves.  How do we state that we have an array-of-strings?  or an array-of-integers?
or an object-with-name-string-and-id-number? or object-with-any-keys-but-integer-values?
  
Systems like JSON-Schema can do this to some extent, but the resulting descriptions are harder to 
work with (read and write) than the data structures. Let's compare JSON Schema, Avro and qb1 for describing
some data types:
  
data sample
                                                                                            
    [ "apple", "orange" ]
    [ 1,2,3 ]
    { "name": "bill", id: 21553 }
    { "ax": 23.1, "ca": 43.3, "gg": 87 }        - values are decimals, but keys can be any string

qb1

    [ "string" ]                                                                                           
    [ "integer" ]                                                                                                      
    { "name": "string", "id": "integer" }                                                          
    { "*": "decimal" }                                                                      

JSON Schema:

    { "type": "array", "items": { "type": "string" } }                                                                 
    { "type": "array", "items": { "type": "integer" } }                                                                            
    { "type": "object", "properties": { "name": { "type": "string", "id": "type": "number" } } }               
    * cannot express 'any-string-key-and-decimal-values' *                                            
                                
Apache Avro

    { "type": "array", "items": "string" }
    { "type": "array", "items": "int" }
    { "type": "record", "fields": { "name": "name", "id": "int" }
    * cannot express 'any-string-key-and-decimal-values' *                                            

    
qb1's is even more concise when using javascript and qb1's brief names:

    [ 'str' ]                                                         
    [ 'str' ]                                                                     
    { name: 'str', id: 'int' }                                
    { '*': 'dec' }                                     


Looking at Schema languages like Avro and JSON Schema we can see that while they are doing a fine job of addressing
form validation, but they aren't helping data scientists and programmers *summarize* and *understand* data.
They also are not as effective as they could be at helping programs *work with data*.  Consider
a path that describes a node in the type graph.  With qb1, the type graph and data graph have the
same structure.  This makes qb1 paths easy to understand and versatile across data selection and type
specification.

Notice how qb1 type definitions mirror the style and simplicity of the data?  This is important for
developers and data scientists who want to *understand* what they are looking at.  It can help scientists
and engineers work more efficiently and make fewer errors.

For many developers, type-safety and validation come to mind first as a major benefits for a type system, but
type-safety was not a driver for the system. We find that unit-tested 
software in a dynamic languages can be simple and effective 
without type safety.  Data form validation wasn't a driver either, though qb1 works for validation.

So why did we do it?  The drivers for an qb1's intuitive type system was to *work more effectively with data*.  We 
work with data more effectively when we can see and work within the boundaries of its structure - and we
can do that when we have a clear and concise way of expression structure.  We
can build more **efficient** systems when these systems have this type information, and developers can construct
all types of handling for data more easily when we can *see* the data structure in an intuitive form.

## A Simple But Comprehensive Type System

A qb1 self-describing data value is of the form:

    { $type: ..., $value: ... }

So a formal way of describing a single string would be: 

     { $type: 'string', $value: 'hi' }                  same as "hi"
    

A number: 

     { $type: 'number', $value: 7 }                     same as 7

An array of integers:

    { $type: ['int'], $value: [ 1,2,3 ] }               same as [ 1,2,3 ]
    
An array of complex records:

    {
        $type: [{
            id: 'int', 
            title: 'str',
            email: 'str'
        }],
        $value: [
            {
                id: 3256,
                title: 'how do we change this function?',
                email: 'abwe@ymail.com'
            },
            {
                id: 2521,
                title: 'is this the correct button?',
                email: 'qwe@ymail.com'
            },
            {
                id: 2985,
                title: 'how can I set the color?',
                email: 'fribby@ymail.com'
            },
            {
                id: 9134,
                title: 'reset still working?',
                email: 'zcase@ymail.com'
            },
            {
                id: 1823,
                title: 'quit button flaking out?',
                email: 'stan@ymail.com'
            },
        ]

While the first two $type/$value examples may seem cumbersome and unnecessary, it is important 
that we establish a consistent form for all
data regardless whether it is a two character string or collection of thirty billion records.  No matter what
it is, we can effectively *describe and store* that data.  Using this basic $type/$value foundation, qb1 gives us very powerful
format choices that can we can use to move and manage our data.

## Types are Values

When designing qb1, I thought long and hard about how to unify concepts into as few components as possible in 
a way that would *extend* the effectiveness of the system.  That is, if someone invests in learning
how to describe and work with data in qb1, describing qb1 custom types should align with that.  Also,
tools that work with an manipulate qb1 data should also work on qb1 type descriptions, because when 
we get really serious about (and exerienced with) type definitions, we find that type definitions, like other data,
needs to be versioned and managed.  It's no different.

Type definitions *are* data and so can be described in our the $type/$value form by using 'type' as the $type.  
So when describing types, these simple and $type/$value forms are equivalent:

    simple form                             $type/$value form
                                            
    'int'                                   { $type: 'type', $value: 'int' }
    [ 'int' ]                               { $type: 'type', $value: [ 'int' ] }
    [ { title: 'str', email: 'str' } ]      { $type: 'type', $value: [ { title: 'str', email: 'str' } ] }
    
We can substitute the type-value form for the simple form at any point within the type definition, 
for example:

    simple form                             nested type/value examples
                                            
    [ { title: 'str', email: 'str' } ]      [ { $type: 'type', $value: { title: 'str', email: 'str' } } ]
    [ { title: 'str', email: 'str' } ]      [ { title: { $type: 'type', $value: 'str' }, email: 'str' } ]

... though unnecessary because in a type context the values above were already known to be of $type 'type' -
however the consistent interchangeability with data objects gives us all the benefits of qb1's efficient
serizliation underpinnings.  What this demonstrates is that qb1 types can be compressed and streamed 
in exactly the same ways that we stream any other data.  Managing types seamlessly as data is an important aspect of
any system where data types change over time.

As a matter of simplicity and conveninence the qb1 type system employs the $-prefix *also* for type 
properties.  These prefixed properties allow qb1 to flatten custom object definitions (a.k.a. records) into
a form that matches the data.  By taking care that our internal data type properties don't 
use the already-taken '$type' and '$value' qb1 allows multiple levels to be combined.  Though we could 
have introduced separate prefixes for type properties, it turns out to be easier on our users to just 
have one '$' prefix for both purposes.

With that in mind, we introduce $base.  The **$base** property allows us to define types based upon another type.

So an anonymous type definition might look like this:

    { $base: 'int', $stip: { range: '0..99' } }

... which is equivalent to just using the string range: 

    '0..99'
    
So for this example, the object form isn't necessary, but it is important to understand the object
form because it works for defining any type, with any number of properities and constraints.  If
we wanted to name our type, we would have to use the object form:
    
    { 
        $base: '0..99',
        $name: 'year21',
        $description: 'year of the 21st century'
    }

The $name property is an identity effective in the scope in which it is defined.  So if this type was defined within
the root typespace '/qb1/...'

    var qb1 = require('qb1').create()       // create an instance of qb1 with modifiable scope
    var qb1.scope.put(
        '/qb1',                             // qb1's global typesapce (in fact refers to the type data series /qb1/type/...)
        { 
            $base: '0..99',
            $name: 'year21',
            $description: 'year of the 21st century'
        }
    )

... then 'year21' would be accessible as /qb1/year21, but also as simply 'year21' because the
'/qb1' namespace is the global namespace included by default:

    qb1.parse("{$type: 'year21', $value: 33}")
    
    > 33

    qb1.parse("{$type: 'year21', $value: 100}")
  
    > error: year21 value is out of range: 100

If we had put the type in scope using:

    var qb1 = require('qb1').create()       // create an instance of qb1 with modifiable scope
    var qb1.scope.put(
        '/myarea',                          // a custom namespace
        { 
            $base: '0..99',
            $name: 'year21',
            $description: 'year of the 21st century'
        }
    )
    
Then we would need to refer to this type using the path '/myarea/year21':
  
    qb1.parse("{$type: 'year21', $value: 33}")
    > error: type year21 is undefined

    qb1.parse("{$type: '/myarea/year21', $value: 33}")
    > 33

Alternatively, we could declare the scope in the root of our object:

    qb1.parse("{$scope: [ '/qb1/*', '/myarea/*' ], $type: 'year21', $value: 33}")
    > 33

If we had stored our type definitions as an array of types:

    var qb1.scope.put(
        '/myarea',                     // a custom typespace
        [ 
            { 
                // version 1
                $base: '0..89',
                $name: 'year21',
                $description: 'year of the 21st century'
            },
            { 
                // version 2
                $base: '0..99',
                $name: 'year21',
                $description: 'year of the 21st century'
            }
        ]
    )

    
Then qb1 would interpret the list as type versions where '/myarea/year21' would refer to the latest 
version (highest index), and adding the version number (starting with 1), would refer to specific versions
'/myarea/year21/1' (version 1 - the first entry)

Then we would need to refer to this type using the path '/myarea/year21':
  
    qb1.parse("{$type: '/myarea/year21', $value: 99}")     // latest version (2)
    > 99
    
    qb1.parse("{$type: '/myarea/year21/1', $value: 99}")   // version 1
    > error: /myarea/year21/1 value is out of range: 99
    
Object form also allows more constraint options, and specification of arbitrarily complex
types.

To understand how qb1 can parse these structures, we can use qb1 itself to explain the how value types are 
always defined as we walk through an arbitrarily complex type (graph).  qb1 type definitions adhere to this definition:
 
    'nul|int|str'
    { $multi: [ nul, int, str ] }
    { $enum: [ 'red', 'green', 'blue' ] }
    {
        $name: 'type',
        $multi: [
            'string', 
            [ 'type' ],
            {
                '^$name': 'string',
                '^$base': 'string',
                '^$desc': 'string',
                '^$array': [ 'type' ], 
                '^$enum': { $multi: [ ['num'], ['str'], ['boo'] ] },    // enums are of a consistent type
                '^$stip': 'stip',           // stipulations are a record structure that add more constraint detail to the type definition
                '/[^$].*/': 'type'          // other properties (non-$) are custom fields
            }
        ]
    }
    
Using this type definition as a guide for parsing, qb1 knows that when parsing a $type, it will be of type 'type' and if
that is an array, then it contains types, and those array types with non-$ fields, those are all types, and those
can be expressed as strings, in this case the string 'str'.

    $type: [ { title: 'str', email: 'str' } ] 

    $type: 'type', where 'type' is [ 'type' ], and that 'type' is { '*': 'type'}, and that 'type' is a string.                  
            

**A note about specificity** 

A more specific type definition could define the restricted *combinations* of properties allowed:

    {
        $name: 'type',
        $multi: [
            'string', 
            [ 'type' ],
            { 
                '^$base': /m|mul|multi/,
                '^$name': 'string',
                '^$desc': 'string',
                '^$multi': [ 'type' ],          
                '^$stip': 'stip'          
            },
            { 
                '^$base': /a|arr|array/,
                '^$name': 'string',
                '^$desc': 'string',
                '^$array': [ 'type' ],
                '^$stip': 'stip'          
            },
            { 
                '^$base': /o|obj|object|n|nul|null/,        // null defaults to 'object' type
                '^$name': 'string',
                '^$desc': 'string',
                '/[^$].*/': 'type', 
                '^$stip': 'stip'          
            },
            { 
                '^$base': /e|enu|enum/,
                '^$name': 'string',
                '^$desc': 'string',
                '^$enum': { $multi: [ ['num'], ['str'], ['boo'] ] },
                '^$stip': 'stip'          
            },
            { 
                '^$base': 'n|num|number|s|str|string...',         // all other types
                '^$name': 'string',
                '^$desc': 'string',
                '^$stip': 'stip'          
            }
        ]
    }

... but this detail is more than we need for a parser to know type context.  Keep this in mind when learning qb1.  
We often want enough information to get a job done without getting overwhelmed with detail.  
A simpler and more general definition can be more effective for understanding and working with data.
qb1 supports very detailed type definitions for form validation *separately* from basic structural information. 

Now since qb1 knows all the fields in an object definition are type 'type', stating so is
redundant, but it's important that the language allows us to excercise self-description of any value at any
point in our data graphs.  The design allows the creation of arbitrary self-describing compositions of any 
value (and types are values - so it applies to any type too).  A powerful 
concept indeed!
    
or number:

    {
        $type: 'number',
        $value: 4
    }


or complex type, such as array-of-strings:

    {
        $type: [ 'string' ],
        $value: [ 'hi there', 'oh hi', 'long time...', 'certainly has been...' ]
    }

or object with different fields:
    
    {
        $type: { 
            id: 'number', 
            title: 'string',
            email: 'string',
            comment: 'string' 
        },
        $value: {
            id: 3,
            title: 'A Change in Approach',
            email: 'fred@gmail.com',
            comment: 'this is an example object'
        }
    }
    
Notice that the field descriptions { id: 'number', title: 'string', ... } are written at 
the same level with the same structure as the data.  This structural similarity between
type and data speeds human comprehension and makes data and types easier to work with.  qb1 uses
the $-prefix to clearly separate both type and meta properties from data leaving the same 
familiar and simple structure for both.  In fact, type object can be flattened again using
$-separation of properties:

    {
        $type: { 
            id: 'number', 
            title: 'string',
            email: 'string',
            comment: 'string' 
        },
        id: 3,
        title: 'A Change in Approach',
        email: 'fred@gmail.com',
        comment: 'this is an example object'
    }

The benefit of flattening becomes clearer when we define our schema in separate (self-described) value and give it 
a name using '$name':

    {
        $type: 'type',
        $value: { 
            $name: 'comment',
            id: 'number', 
            title: 'string',
            email: 'string',
            comment: 'string' 
        }
    }
    
Again, we can flatten the nested type description, pulling the $value properties into the parent:

    var comment_type = {
        $type: 'type',
        $name: 'comment',
        
        id: 'number', 
        title: 'string',
        email: 'string',
        comment: 'string' 
    }
    
That's our object schema as a flat structure.  We can add the type to a typeset like so:
    
    var typeset = require('qb1-typeset')
    typeset.add( comment_type )

... and then refer to it in a self-described object:
can put $type and $name

    {
        $type: 'comment_type',
        $value: {
            id: 3,
            title: 'A Change in Approach',
            email: 'fred@gmail.com'
            comment: 'this is an example object'
        }
    }

... again, we can flatten $value into the data object:

    {
        $type: 'comment_type',
        id: 3,
        title: 'A Change in Approach',
        email: 'fred@gmail.com'
        comment: 'this is an example object'
    }

## So why do we care about flattening type objects and symmetry?

Because it is so much more intuitive to work with type definitions that mirror the data
they represent.  Time is saved, fewer mistakes made, less clutter.  Benefits are clear when data
types get more complex.  Let's describe an collection of comment objects 'topic_comments' to show this:

    {
        $type: {
            $name: 'topic_comments',
            
            id: 'number',
            title: 'string',
            comments: [
                {
                    email: 'string',
                    comment: 'string'
                }
            ]
        },    
        $value: {
            id: 3,
            title: 'A Change in Approach',
            comments: [
                {
                    email: 'fred@gmail.com'
                    comment: 'great.  just great.'
                },
                {
                    email: 'sara@customwidgets.com',
                    comment: 'not a fan.  not yet.'
                }
            ]
        }
    }
    
We can quickly understand the types and data because are the same structure.  If we didn't allow 
type $-property hoisting, we would have to put type field descriptions under another
'fields' property - which is what other type systems do today and winds up looking nothing 
like the data itself.  It's tricky to work with:

    {
        type: {
            name: 'topic_comments',
            type: 'object',
            fields: {
                id: 'number',
                title: 'string',
                comments: {
                    type: 'array',
                    value_types: [
                        {
                            type: 'object',
                            fields: {
                                email: 'string',
                                comment: 'string'
                            }
                        }
                    ]
                }
            }
        },    
        value: {
            id: 3,
            title: 'A Change in Approach',
            comments: [
                {
                    email: 'fred@gmail.com'
                    comment: 'great.  just great.'
                },
                {
                    email: 'sara@customwidgets.com',
                    comment: 'not a fan.  not yet.'
                }
            ]
        }
    }
    
The structural similarity is made possible by type property 'hoisting', and can save time and 
headaches when working with data definitions and contents.

## $-Properties 

## Universal Property: $type

The '$type' property is common to type descriptions all values.  It may be a:
 
* string:
  
  as a string, the type can be 
  
  * the name of a built-in type 'string', 'number'....
  * the name of a type loaded into global scope: 'comment', 'my_address'....
  * the path of a type file or set of files in a folder structure in a location registered with a typeset.
  * the path to an inner sub-structure within a type
  * a type expression that resolves to an enumeration or constrained type such as 'string|null' or '0..10'
   
* object:
    
  A type-definition object with any of the type properties described here.  The object form
  can describe any type including objects, primitives, arrays, and enumerations.
  
* array

  A simple way to define array types as a single or cycling types
   
## Universal Properties

Here is a list of qb1 properties that can appear with any type/value object

### $type (required)

A description of the $value data structure which may specify any amount of constraints etc.  Types
are one of the most flexible and powerful aspects of qb1 and can be expressed as the name
of a basic type, a reference to a custom type, or an object or array that define a custom type.

### $value (required)

A value adhering to the structure and constraints defined in $type

### $name 

An short name that serves as an identifier.  **snake_case** is recommended, since
these names can be used to create path references into type structures that can be saved into files or file
structures.

### $description 

An optional description of a value

### $stats

A collection of information written out by qb1 during serialization 
that can be used to understand and optimize data handling. 

### $index

A collection of offsets that qb1 may write to help speed up data traversal and searching.

## Type Definition Properties

The following properties are specific to type definitions using the object construct.

### $tinyname

In addition to the $name, a very small name can be given to some types.  This is recommended only
if when a type is referred to heavily such as a core type.

### $fullname

An optional longer name that can be used to create more descriptive output.

### $stipulations

Optional constraints that can be added to a type definition, requiring properties and 
constraining values to ranges etc.

### $arr

For arrays types, this property defines the cycling array of types that the values must match, so an array with a
single type:

    $type: { $base: 'arr', $name: 'int_list', $arr: [ 'int' ] }      // note that $base is optional here.  see below.
    
Describes values:

    [ 1 ]
    and
    [ 1, 2, 3 ]

While a three type cycle: 

    [ 'string', 'integer', 'boolean' ]
    
Describes arrays such as:

    [ 'a', 1, true ]
        and
    [ 'a', 1, true, 'b', 17, true, 'c', 27, false ] 
    
But not:

    [ 1, true, 'a' ]
        or
    [ 'a', 1, true, 'b' ]
    
... because they do not fully match the cycle of types.  Note that to match a list of possible types such 
as null-or-string-or-integer, one can use a pipe string expression 'nul|str|int' or use a 'multi' type.

As a convenience, the $arr property defaults the $base to 'arr', so one can simply write:
 
    $type { $name: 'int_list', $arr: [ 'int' ] }
    

### $mul

For multi types, this property holds the list of all the many types that the value may match.
    
    {
        $type: { $mul: [ 'str', 'nul', [ 'int' ] ] },
        $value:  null
    }

### Other Fields (object fields and patterns)

All fields without a dollar prefix are intepreted as field type definitions using

    name: type
    pattern: type, 
    
... where name is a simple string, and pattern is a either a simple pattern containing one or more
wild cards ('*') or a slash-bounded regular expression /.../

It's best to avoid overlap with the $ namespace, but know that if you must specify a custom field 
that begins with '$' such as '$name' one can use the carrot-$ escape:

    '^$name': 'string'
    
... defines a custom string field '$name', just like our built-in.

    
# OLDER SECTION - NEEDS REVISION

## Types as Self-Described Values

The address type description above can be written as a stand-alone, self-describing object.  Here 
is a description with some zip code constraints ('stipulations') thrown in:

    {
        $type: 'type', 
        $value: { 
            $base: 'object',
            $fields: {
                street: 'string',
                city: 'string',
                state: 'string',
                zip: 'number'
            }
            $stipulations: {
                zip: { range: '1..99999' }
            }
        }
    }
    
This format follows the similar $type... $value format as qb1 data with one important
difference: the type properties, such as 'base' and 'fields' use the dollar ($) prefix.  This
allows types 

While clear and consistent, the basic $type...$value form gets complicated when we have nested types.
Let's create some clutter in our type definition by imagining that address 'state' is actually a 'states' field
which is an array of a objects, not just a string (goofy data choice, but let's just go with it)

    {
        $type: 'type',
        $value: { 
            $base: 'object',             
            $fields: {
                street: 'string',
                city: 'string',
                zip: 'number',
                states: {
                    $type: 'type',
                    $value: {
                        $base: 'array',
                        $items: [
                            {
                                $type: 'type',
                                $value: {
                                    $base: 'object',
                                    $fields: {
                                        name: 'string',
                                        abbreviation: 'string',
                                        acquired: 'number'
                                    }
                                }
                            }
                        ]
                    }
                }
            },
            stipulations: {
                $type: 'stipulations',
                $value: {
                    zip: { range: '1..99999' }
                }
            }
        }
    }

That is valid qb1 syntax, but it is overly complex.  If we parsed this with 
qb1 and printed it out, it would print like the .  Here's how:

First we can **hoist all the contents of each type $value: {...} out to the $value itself**
The $-prefix on type properties makes this possible - it's actually the reason why qb1 has the prefix.

    {
        $type: 'type', 
        $base: 'object',
        $fields: {
            street: 'string',
            city: 'string',
            zip: 'number',
            states: {
                $type: 'type',
                $base: 'array',
                $items: [
                    {
                        $type: 'type',
                        $base: 'object',
                        $fields: {
                            name: 'string',
                            abbreviation: 'string',
                            acquired: 'number'
                        }
                    }
                ]
            }
        },
        $stipulations: {
            $type: 'stipulations',
            zip:               { range: '1..99999' },
            'states/acquired': { range: '1000..9999' },
        }
    }
    
Notice how the $-prefix helps the reader quickly distinguish custom object fields
from type properties like '$base' and '$stipulations'.

Next we take advantage of two qb1 simplification features

1. array types like this:
   
       { 
           $type: 'type', 
           $base: 'array', 
           $items: [ 'type-a', 'type-b'... ] 
       }
   
   Can be defined using a simple array of type-names:
   
       [ 'type-a', 'type-b'... ]
    
1. object types :
   
        { 
            $prop1: val1, 
            $prop2: val2, 
            $base: 'object', 
            $fields { 
                a: 'type-a', 
                b: 'type-b' 
            }
         }
    
    can be written as flat objects, with the $-prefix serving to separate meta properties
       from the field names:
    
        { 
            $prop1: val1, 
            $prop2: val2, 
            a: 'type-a', 
            b: 'type-b' 
        }


Using these simplifications and adding a $name to our addresses example, we get:

    {
        $type: 'type',                                  //  meta-property
        $name: 'address',                               //  meta-property
                                                        
        street: 'string',                               //  
        city: 'string',                                 // 
        zip: 'number',                                  // 
        states: [                                       // 
            {                                           // 
                name: 'string',                         //   hoisted $value.fields
                abbreviation: 'string',                 // 
                acquired: 'number'                      // 
            }                                           // 
        ],                                              // 
        
        $addenda: {
            range:
                'zip':                 '1..99999',
                'states/acquired':     '1000..9999'
            }
        }
    }
    
... which is the same structural shape as the data format.  

And here's how we can write this self-described data as an anonymous $type...$value object, putting
the above type declaration under another $type property - and removing $name and $type, which
are not needed (though they could be left if desired).

    {
        $type: {                                           //  type
                                                           //
            street: 'string',                              //        //  
            city: 'string',                                //        // 
            states: [                                      //        // 
                {                                          //        // 
                    name: 'string',                        //        //   hoisted type.$value.fields
                    abbreviation: 'string',                //        // 
                    acquired: 'number'                     //        // 
                }                                          //        // 
            ],                                             //        // 
            zip: 'number'                                  //        // 
                                                           //        
            $add: {
                range:
                    'zip':                 '1..99999',
                    'states/acquired':     '1000..9999'
                }
            }
        }
        $value: {                                          //  value
            street: 55 Main St,                            // 
            city: Tuscon,                                  // 
            states: [                                      // 
                {                                          // 
                    name: Arizona,                         // 
                    abbreviation: AZ,                      // 
                    acquired: 1912                         //    
                },                                         // 
                {                                          // 
                    name: California,                      // 
                    abbreviation: CA,                      // 
                    acquired: 1848                         // 
                }                                          //
            ],                                             // 
            zip: 85701                                     // 
        }
    }
    
If you have stored the type definition 'address' type in a catalog (more on that later), 
you can use the $name reference for the $type like so:


    {
        $type: 'my_types/address/1',
        $value: {
            street: 55 Main St,
            city: Tuscon,
            states: [
                {
                    name: 'Arizona',
                    abbreviation: 'AZ',
                    year_founded: 1912
                },
                {                                          
                    name: California,                      
                    abbreviation: CA,                      
                    acquired: 1848                         
                }                                         
                
            ],
            zip: 85701
        }
    }
    
Similar to the way we can hoist type definition $value/fields, data objects can have their properties
hoisted outside of the $value field to flatten the objects:

    {
        $type: 'my_types/address/1',
        street: 55 Main St,
        city: Tuscon,
        states: [
            {
                name: 'Arizona',
                abbreviation: 'AZ',
                year_founded: 1912
            },
            {                                          
                name: California,                      
                abbreviation: CA,                      
                acquired: 1848                         
            }                                         
        ],
        zip: 85701
    }

That's the concise data form.  It should look familiar since it follows the same convention as
the concise type definition: 

    {
        $type: 'type',                                  //  meta-property
        
        street: 'string',                               //  
        city: 'string',                                 // 
        states: [                                       // 
            {                                           // 
                name: 'string',                         //   hoisted $value.fields
                abbreviation: 'string',                 // 
                acquired: 'number'                      // 
            }                                           // 
        ],                                              // 
        zip: 'number'                                   // 
    }

    
So in qb1, types are values and values have types and all can be expressed concisely and intuitively: an 
important symmetry with some powerful implications we will demonstrate later on.

### Types Are Values

The inter-changability of types and values can be shown with a substitution of a basic type.  Taking
the (albeit strange) multi-state address type definition from before

    {
        $type: 'type',                                  

        street: 'string',                                 
        city: 'string',                                  
        states: [                                        
            {                                            
                name: 'string',                         
                abbreviation: 'string',                  
                acquired: 'number'                       
            }                                            
        ],                                               
        zip: 'number'                                    
    }

We can treat built-in types like 'string' and 'number' and [...] (array) just like custom types and expand them into
$type...$value form:

    {
        $type: 'type',   
        street: {
            $type: 'type',
            $value: 'string'
        }
        city: { 
            $type: 'type',
            $value: 'string'
        }
        states: {
            $type: 'type',
            $value: [                                        
                {                                            
                    name: 'string',                         
                    abbreviation: 'string',                  
                    acquired: 'number'                       
                }                                            
            ],
        }                                               
        zip: {
            $type: 'type', 
            $value: 'number'
        }
    }

... anything placed into $value is handled exactly like would be without type/value specifiers.  Why
would we do something like this?  Well, basic types are open to structuring with metadata just like
any other, and can be constrained and managed exactly the same.



