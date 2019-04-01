# Comment  8/30/17

The use of "$type" here isn't really correct.  "$type" : "string" is already used 
for *values* so it is confusing to use it as a type definition just because we are in type context.

see "complex-types.md" for the new thinking on this.



# qb1-schema

These examples documents were moved here from the qb1-schema project

# JSON Schema
```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "required": [ 
    "id", 
    "body", 
    "tags" 
  ],
  "properties": {
    "id": {
      "type": "integer"
    },

    "body": {
      "type": "string"
    },

    "tags": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "maxItems": 6
    },

    "references": {
      "type": "array",
      "items": {
        "type": "string",
        "format": "uri"
      }
    }
  }
}
```

# qb1-schema

We can represent the same information in qb1-schema with:

```
{
    "id": "int",
    "body": "str",
    "tags": "[ str, 0..6 ]",
    "references": "[ uri ]",
    "$require": "[ id, body, tags ]" 
}
```

But let's break the syntax down so the rules are understood.  qb1 schemas can be written much like 
JSON schemas in expanded form:
```
{
    "id": {
        "$type": "integer"
    }
    "body": { 
        "$type": "string"
    }
    "tags": {
        "$type": "array",
        "$vtype": "string",
        "$length": "0..6"
     },
    "references": {
        "$type": "array",
        "$vtype": "string"
    ],
    "$require": [ 
        "id", 
        "body", 
        "tags" 
    ]
}
```

Type attributes may be expanded in this way similar to JSON Schema using dollar-attributes "$" (the dollar 
differentiates attributes of the type itself from custom attributes). 

But since qb1 accepts simple strings, we don't have to use objects with dollar attributes for each
definition, we can use strings:

    { "$type": "string" }       same as     "string"
    { "$type": "integer" }      same as     "integer"
    
For objects and arrays, instead of:

    { "$type": "array", "$vtype": "string", "$length" }        (an array containing only string values)
    { "$type": "object", "$vtype": "integer" }      (an object containing only integer values)
    
... we can use JSON braces {...} and brackets [...]:

    [ "string" ]
    { "$vtype": "string" }
    
Notice that with arrays, we use positional arguments and with objects, we use key/value to fit with 
JSON form.

To define the custom object with different typed fields, we use natural (non-dollar) string keys 
for each field.  The second array argument is the length range:

    {
        "id": "integer",
        "body": "string",
        "tags": [ "string", "0..6" ],
        "references": [ "uri" ]
    }

In qb1, the tags array definition can be written using brackets instead of "$type":

    "tags": [ "$vtype": "string", "$length": "0..6" ] 
    
... since the bracket '[' itself indicates that this is an array type.
 
Going a step further, qb1/array defines array properties type, vtype, length as ordered properties, 
so we can also just write: 
 
    "tags": [ "string", "0..6" ]
    
Reducing further, we can use qb "plain" format for the definition by placing the entire expression
in quotes and removing the inner quotes:

    "tags": "[ string, 0..6 ]"
    
Finally , qb1 recognizes short names for all the common types, so we can write this definition as simply:
```
{
    "id": "int",
    "body": "str",
    "tags": "[ str, 0..6 ]",
    "references": "[ uri ]",
    "$require": "[ id, body, tags ]" 
}
```

If we have this schema locally saved at './my_qb1/type/entry.qb1', we can read it with:
 
    var qb1 = require('qb1').init('http://quicbit.com/qb1/v4, './my_qb1')
    
    var entry_type = qb1.read('type/entry')

Notice that qb1 was initialized with a url and path source which set how qb1 resolves type names such
as "uri".  Schemas are resolved in order, so if we had reversing the source paths:

    var qb1 = require('qb1').init('./my_qb1', 'http://quicbit.com/qb1/v4')
    
...then types in '.my_qb1' could override core types, which is not usually desired, but can be done.

## Named Types



## qb1 Long, Normal and Short Form

qb1 recognizes full, short and very-short names.  Short names are used by default and simply referred
to as "name".  

    qb1.write(type)
     
outputs:

```json
{
  "$typ": "typ",
  "$dat": {
    "id":         "int",
    "body":       "str",
    "tags":       "[str, 0..6]",
    "references": "[uri]"   
  },
  "$req": "[id,body,tags]"
}
```

    qb1.write(type, { name: 'full' }
    
outputs:
```json
{
  "$type": "type",
  "$data": {
    "id":         "integer",
    "body":       "string",
    "tags":       "[string, 0..6]",
    "references": "[uri]"   
  },
  "$req": "[id,body,tags]"
}
```

    qb1.write(type, { name: 'short' })
    
... outputs a super-short version, which can be useful for some types of serialization work:

```json
{"$t":"t","$d":"{id:i,body:s,tags:[s,0..6],references:[url]},$req:[id,body,tags]}"}
```

## Expanded Form

The qb1 form given above is actually a short-hand.  qb1 supports shorthand for the most
common attributes and settings.  Here's the schema using expanded form:
 
```json
{
  "$type": "type",
  "$data": {
    "id":         { "$type": "integer" },
    "body":       { "$type": "string" },
    "tags":       { "$type": "array", "$valtype": "string", "$length": "0..6" },
    "references": { "$type": "array", "$vt": "uri" }
  },
  "$require": "[id,body,tags]"
}
```

And here it is, still expanded, but using the equivalent short names and expressions:

```json
{
  "$typ": "typ",
  "$val": {
    "id":         { "$typ": "int"},
    "body":       { "$typ": "str"},
    "tags":       { "$typ": "arr", "$vt": "str", "$len": "..6" },
    "references": { "$typ": "arr", "$vt": "uri" }
  },
  "$req": "[id,body,tags]"
}
```

The dollar ($) attributes are attributes of the type itself.  We can add a name to our type object
with the $name attribute:

```json
{
  "$type": "type",
  "$data": {
    "$name":      "entry",
    "id":         { "$typ": "int" },
    "body":       { "$typ": "str" },
    "tags":       { "$typ": "arr", "$vt": "str", "$len": "..6" },
    "references": { "$typ": "arr", "$vt": "uri" },
    "$require":   "[id,body,tags]"
  }
}
```

The $name attribute allows us to reference this type by the given name.

## Plain String Form and Intercompatibility with JSON

While qb1 is 100% compatible with JSON, JSON is not the most readable format for qb1.  qb1 in 
plain string form can be easier to work with when writing tests and scripts because
it eliminates unneeded quotes:

### JSON Format

```
{
    "$name": "entry",
    "id": "int",
    "body": "str",
    "tags": [ 
        "str", 
        "0..6" 
    ],
    "references": [ 
        "uri" 
    ],
    "$require": [ 
        "id", 
        "body", 
        "tags" 
    ]
}
```

### Plain Form 

In plain string form, quotes are optional and may be single or double.  

For illustration, if we want to change the name to "the entry", with double-quotes, we
can do this easily in plain form:

```
{
    $name: '"the entry"',
    id: int,
    body: str,
    tags: [ 
        str, 
        0..6 
    ],
    references: [ 
        uri
    ],
    $require: [ 
        id, 
        body, 
        tags 
    ]
}
```

### JSON and Plain In Combination

qb1 seemlessly translates between these forms and can use any combination, which can improve
with conciseness and certain aspects of readability.

JSON with plain strings

```
{
    "$name":        "entry",
    "id":           "int",
    "body":         "str",
    "tags":         "[ str, 0..6 ]",
    "references":   "[ uri ]",
    "$require":     "[ id, body, tags ]"
}
```

## Constraints or Stipulations

qb1 supports concise syntax for most constraints

# Using Short-Names


# References


```json

{
  "$schema": "http://json-schema.org/draft-04/schema#",

  "definitions": {
    "address": {
      "type": "object",
      "properties": {
        "street_address": {
          "type": "string" 
        },
        "city": { 
          "type": "string" 
        },
        "state": { 
          "type": "string" 
        }
      },
      "required": [
        "street_address", 
        "city", 
        "state"
      ]
    }
  },

  "type": "object",

  "properties": {
    "billing_address": { 
      "$ref": "#/definitions/address" 
    },
    "shipping_address": { 
      "$ref": "#/definitions/address" 
    }
  }
}
```

qb1-schema (in JSON)

We can define these types in one file, but for a simple introduction to references, we will begin with
one file for each type, and place them in a directory "./my_qb1/type":



```
{
    "$type": "[type]"
    "$data": [
        {
            "$name":            "address",
            "street_address":   "str",
            "city":             "str",
            "state":            "str",
            "$require":         "[ street_address, city, state ]"
        },
        {
            "$type": "address",
            "$name": "billing_address",
        },
        {
            "$type": "address",
            "$name": "shipping_address"
        }
    ]
}

```

It is worth mentioning that the references to 'address' used for billing_address and shipping_address
work because address is a type (first in an array of types)
