# Example

To understand some of the benefits of qb-schemas, walk through one of the JSON Schema examples 
(from http://json-schema.org/example1.html)

The JSON Schema Example starts with a data for a simple product:

    {
        "id": 1,
        "name": "A green door",
        "price": 12.50,
        "tags": ["home", "green"]
    }

... and creates this schema to represent it...

## JSON Schema

    {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "title": "Product",
        "description": "A product from Acme's catalog",
        "type": "object",
        "properties": {
            "id": {
                "description": "The unique identifier for a product",
                "type": "integer"
            },
            "name": {
                "description": "Name of the product",
                "type": "string"
            },
            "price": {
                "type": "number",
                "minimum": 0,
                "exclusiveMinimum": true
            },
            "tags": {
                "type": "array",
                "items": {
                    "type": "string"
                },
                "minItems": 1,
                "uniqueItems": true
            }
        },
        "required": ["id", "name", "price"]
    }

Notice the difference between the data structure and the schema.  The data is simpler.  Can
we can make the description simpler and more like the data itself?  qb-schemas do just that,
they represent the data definition in much the same way as the data itself.

## qb-schema

Let's model this data again:

    {
        "id": 1,
        "name": "A green door",
        "price": 12.50,
        "tags": ["home", "green"]
    }

But with qb-type, we mirror the data in the type definition:

    {
        "id": "integer",
        "name": "string",
        "price": "number",
        "tags": [ "string" ]
    }

The brackets themselves express we have an object '{..}' containing fields "id" (an integer), "name" (a string),
"price" (a number), and "tags" (an array of string).
    
This structure may be all we need to process data, but for this example, we want to add constraints to
"price" and to the "tags" fields.  Let's add constraints to the "number" type with
**parenthesis arguments**, which work much like function arguments: "number(arg1, arg2, ...)".  We pass
number the range argument "0~~" to limit prices to numbers above zero.

    {
        "id": "integer",
        "name": "string",
        "price": "number(0~~)",
        "tags": [ "string" ]
    }

'0~~' is qb range notation for an open-open interval between
zero and javascript's Number.POSITIVE_INFINITY.  We can also express this decimal type and constraint as just: 
    
    "price": "0.0~~" 

See ranges.md for more about ranges.

Now let's constrain the array for "tags".  We have two choices.  All types have an
object form.  <code>[ "string" ]</code> 
is really just a short-hand for: { "$type": "array", "$items": [ "string" ] }.  The advantage of
object form, is that we can set any type property using the '$' prefix.  So our type structure
becomes:

    {
        "id": "integer",
        "name": "string",
        "price": "number(0~~)",
        "tags": { 
            "$type": "array", 
            "$length": "1..", 
            "$uniq": true, 
            "$items": [ "string" ] 
        }
    }

Perfectly acceptable, but the downside of this approach is that it takes us away from the 
structural similarity between types and original data. 

An alternative that keeps structural similarity is to specify constraints in the $stipulations 
field of the containing object.  Let's do this for price and tags:

    {
        "id": "integer",
        "name": "string",
        "price": "number",
        "tags": [ "string" ],
        
        "$stipulations": {
            "price": "0~~",
            "tags": { "length": "1..", "uniq": true }
        }
    }

When working with large and complex nested structures, the separation of constrain information
can save time and headaches when working with data.

Now, let's continue following the JSON Schema example by adding new nested properties "dimensions" and "warehouse" and 
defining an array of products.  So if the data is in this form:

    [
        {
            "id": 2,
            "name": "An ice sculpture",
            "price": 12.50,
            "tags": ["cold", "ice"],
            "dimensions": {
                "length": 7.0,
                "width": 12.0,
                "height": 9.5
            },
            "warehouseLocation": {
                "latitude": -78.75,
                "longitude": 20.4
            }
        },
        {
            "id": 3,
            "name": "A blue mouse",
            "price": 25.50,
            ...
        }
    ]
    
... we can simply wrap our definition in array-brackets and add new definitions in similar structure:

    [
        {
            "id": "integer",
            "name": "string",
            "price": "number",
            "tags": [ "string" ],
            "dimensions": {
                "length": "number",
                "width": "number",
                "height": "number"
                "$stipulations": {
                    "length": "0..",
                    "width": "0..",
                    "height": "0..",
                }
            },
            "warehouse_location": {
                "latitude": "number",
                "longitude": "number"
                "$stipulations": {
                    "latitude": "0..",
                    "logitude": "0.."
                }
            }
            "$stipulations": {
                "price": "0~~",
                "tags": { "length": "1..", "uniq": true }
            }
        }
    ]

That's the general way to construct any type, but note that qb1 supports greater 
flexibility and conciseness.  For example, we can move stipulations outside of the inner 
objects with path references, use numeric ranges as shorthand for type/stipulation pairs, 
and use short-hand names.

    [
        {
            "id": "int",
            "name": "str",
            "price": "0.0~~",
            "tags": [ "str" ],
            "dimensions": {
                "length": "0.0..",
                "width": "0.0..",
                "height": "0.0.."
            },
            "warehouse_location": {
                "latitude": "num",
                "longitude": "num"
            }
            "$stipulations": {
                "tags": { "length": "1..", "uniq": true }
            }
        }
    ]

With type definition and data so similar, you can quickly define types definitions by 
simply taking a data sample and replacing values with type descriptors.

For the final step, we define the product reference ($name) and required properties ($required) 
in the product 
and dimensions objects.  Objects implied by braces notation "{...}" (i.e. 
the product, dimensions and warehouse_location objects) are extremely 
similar to expanded form using the "$type" property - in fact, these objects **are in expanded form** 
with two convient differences:

  1.  The "$type" is set to "object" by default, so there is no need to specify it.
  2.  Non-dollar properties are interpreted as "$fields" values, which are custom type definitions
      like JSON Schema "properties". 
  
qb1 uses the "$name" property to create *relative* \* type references and the "$require"
property on objects to require fields, so we can complete our definition with:

    [
        {
            "$name": "product",

            "id": "int",
            "name": "str",
            "price": "0.0~~",
            "tags": [ "str" ],
            "dimensions": {
                "length": "0.0..",
                "width": "0.0..",
                "height": "0.0.."
            },
            "warehouse_location": {
                "latitude": "num",
                "longitude": "num"
            }
            
            "$stipulations": {
                "$require":  [ "id", "name", "price", "dimensions/length", "dimensions/width", "dimensions/height" ] 
            }
        }
    ]

(\* relative references are resolved against the initialized base path(s) or url(s) chosen - they
are more flexible, but just as reliable as absolute references.  That is: reliable-base + relative-name = reliable-uri)

Let's check this schema against our data:

    [
        {
            "id": 2,
            "name": "An ice sculpture",
            "price": 12.50,
            "tags": ["cold", "ice"],
            "dimensions": {
                "length": 7.0,
                "width": 12.0,
                "height": 9.5
            },
            "warehouseLocation": {
                "latitude": -78.75,
                "longitude": 20.4
            }
        }
    ]

Does it look correct?

Here's the JSON Schema, which spells out everything simply and clearly (we are not criticising it!) - 
but just want to show how qb-schemas allow better data-mirroring, if you are into that sort of thing.
For large and complex schemas, we find the similar and simpler structure to be helpful.

JSON Schema 

    {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "title": "Product set",
        "type": "array",
        "items": {
            "title": "Product",
            "type": "object",
            "properties": {
                "id": {
                    "description": "The unique identifier for a product",
                    "type": "number"
                },
                "name": {
                    "type": "string"
                },
                "price": {
                    "type": "number",
                    "minimum": 0,
                    "exclusiveMinimum": true
                },
                "tags": {
                    "type": "array",
                    "items": {
                        "type": "string"
                    },
                    "minItems": 1,
                    "uniqueItems": true
                },
                "dimensions": {
                    "type": "object",
                    "properties": {
                        "length": {"type": "number"},
                        "width": {"type": "number"},
                        "height": {"type": "number"}
                    },
                    "required": ["length", "width", "height"]
                },
                "warehouseLocation": {
                    "description": "Coordinates of the warehouse with the product",
                    "$ref": "http://json-schema.org/geo"
                }
            },
            "required": ["id", "name", "price"]
        }
    }
    
    
Wait!  Where's the title?  Where's the shema url?  Well, title isn't needed because we can create our
own object type with required title property use that to catalog your types. 
As for the schema url, that 
is replaced by the base path(s) or url(s) you point to when you initialize the qb1 library.  We don't think it 
should be bound to the type definition.  Keeping separate allows us to create and share versioned type sets 
independently of the contents of those type definitions, and using unix-like path resolution provides 
great flexibility in testing and configuration.


# Recap:

1.  qb type structures mirror the data structure they define:
    
        { 
            "a": [ 
                { "name": "foo", "id": 3 }, 
                { "name": "bar", "id": 7 } 
            ],
            "b": [ 3.2, 12.8, 27.4 ]
        }
        
        can be defined as:
        
        { 
            "a": [ 
                { "name": "string", "id": "number" }, 
                { "name": "string", "id": "number" }
            ],
            "b": [ "number" ]
        }
        
1.  String type definitions accept parenthesis arguments, which for numbers may be a range notation.

        "price": "number(0~~)"

1.  Array bracket "[...]" definitions accept object or positional arguments following the item-type (which must be first):
        
        "tags": [ "string", { "$length": "1..", "$uniq": true } ]
        
        or
        
        "tags": [ "string", "1..", "uniq" ]
        
    (note that when converting boolean properties to ordered properties, we use the property name. 
    Also, when these properties are not in conflict with argument values, they can be taken in any order)
        
1.  Object braces "{...}" definitions without the "$type" property will interpret $dollar keys 
    as properties defining the object and non-dollar keys as custom property or "fields" definitions:
    
        "dimensions": {
            "length": "number",
            "width": "number",
            "height": "number",
            "$require": [ "length", "width", "height" ]
        }
        
1.  All types can be defined in expanded form using object braces "{...}" and the "$base" 
    property, which is the type upon which the type is based (i.e. extends).  
    
    For example, the width number property could be written in expanded form as:
    
        "width" : { "$base": "number", "$range": "0.0.." }
        
    If we named this number type using "$name", we could refer to it by name elsewhere:
    
        "width" : { "$name": "posnum", "$base": "number", "$range": "0.0.." },
        "height" : "posnum",
        ...
    
    Actually, the format is consistent with self-described data objects.  The "$type" is 
    
        
    
    With expanded form, any supported type property can be set using the dollar prefix.  The
    shorter object form (above) is really the same thing, but defaulting "$type" to "object" and 
    treating plain (non-dollar) keys as entries in the "$fields".  Compare this with above:

        "dimensions": { 
            "$fields": {
                "length": "number",
                "width": "number",
                "height": "number"
            }
            "$require": [ "length", "width", "height" ]
        }
        
        "tags": { 
            "$type": "array",
            "$length": "1..",
            "$uniq": true
        }
        
        "price": {
            "$type": "number",
            "$range": "0~~"
        }

