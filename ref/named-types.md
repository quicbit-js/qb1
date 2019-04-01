# Named Types

As explained in the "named-types-example.md", properties can be set by using object braces "{...}" and dollar-prefix for properties.  Here, 
"$name" and "$require" are properties of the type, while "id", "name", "price", and other plain
properties define the properties or "fields" allowed in objects of this type.


    {
        "$name": "product",
        "id": "integer",
        "name": "string",
        "price": "number(0~..)",
        "tags": [ "string", "1..", "uniq" ],
        "dimensions": {
            "length": "number",
            "width": "number",
            "height": "number",
            "$require": [ "length", "width", "height" ]
        },
        "warehouse_location": {
            "latitude": "number",
            "longitude": "number"
        },
        "$require": [ "id", "name", "price" ]
    }


Let's say we wanted to create a "dimensions" and "location" type and use them in our example.  We 
want to name these two types using the "$name" property:

    {
        "$name": "dimensions",
        "length": "number",
        "width": "number",
        "height": "number",
        "$require": [ "length", "width", "height" ]
    }
    
    {
        "$name": "location",
        "latitude": "number",
        "longitude": "number"
    }

And then refer to them by name in our "product" type:

    {
        "$name": "product",
        "id": "integer",
        "name": "string",
        "price": "number(0~..)",
        "tags": [ "string", "1..", "uniq" ],
        "dimensions": "dimensions",
        "warehouse_location": "location",
        "$require": [ "id", "name", "price" ]
    }


## Defining a File with Many Types

So... where do we put these things?  How do we refer to them?  One option is to place them in 
a typeset file.

    {
        "$type": "[type]",
        "$items": [
            {
                "$name": "dimensions",
                "length": "number",
                "width": "number",
                "height": "number",
                "$require": [ "length", "width", "height" ]
            },
            {
                "$name": "location",
                "latitude": "number",
                "longitude": "number"
            },
            {
                "$name": "product",
                "id": "integer",
                "name": "string",
                "price": "number(0~..)",
                "tags": [ "string", "1..", "uniq" ],
                "dimensions": "dimensions",
                "warehouse_location": "location",
                "$require": [ "id", "name", "price" ]
            }
        ]
    }
    
If the above was in a file called './my_types', the we could include this file as a path in 
the call to qb1.init() using: 
    
    var qb1 = require('qb1').init('./my_types')

We can then get this type information with 

    var product_type = qb1.type('product')
    
## Defining Types in Separate Files

Alternatively, we could store the types in separate files.

In a file ./my_types/dimensions:

    {
        "$type": "type",
        "$name": "dimensions",
        "length": "number",
        "width": "number",
        "height": "number",
        "$require": [ "length", "width", "height" ]
    }
    
In a file ./my_types/location

    {
        "$type": "type",
        "$name": "location",
        "latitude": "number",
        "longitude": "number"
    }
    
In a file ./my_types/product

    {
        "$type": "type",
        "$name": "product",
        "id": "integer",
        "name": "string",
        "price": "decimal(0~..)",
        "tags": [ "string", "1..", "uniq" ],
        "dimensions": "dimensions",
        "warehouse_location": "location",
        "$require": [ "id", "name", "price" ]
    }

Then again, initialize qb1 with the same call.  Even though this time "./my_types" is a directory
and not a file, the call is exactly the same as with the typeset file, above.

    var qb1 = require('qb1').init('./my_types')
    var product_type = qb1.type('product')


## Categorizing and Organizing Types

The absolute definition of types "dimensions" and "location" in the example above is only suitable
for isolated type sets.  Shared type sets may require name-space hierarchies and separation.  qb1
creats type namespaces using "/" in path names, but can interpret this separator consistently
across a number of contexts:

* URI
* Nested Objects (within file)
* File Directories

Instead of "location" and "dimensions", we could have used more specific names.  Let's say we
categorize our names with a "catx" prefix.  So we want our product to look like this:

    {
        "$name": "product",
        "id": "integer",
        "name": "string",
        "price": "number(0~..)",
        "tags": [ "string", "1..", "uniq" ],
        "dimensions": "catx/dimensions",
        "warehouse_location": "catx/location",
        "$require": [ "id", "name", "price" ]
    }

To do this, we leave the files as they are and initialize qb1 using a prefix like so:

    var qb1 = require('qb1').init({ prefix: 'catx', src: './my_types' })
    var product_type = qb1.type('catx/product')
    
Keep in mind that product and it's dependencies can be placed in separate sources and given 
any prefix needed and references are resolved in path order, so the flexibility is there to 
name and override types as needed.  If we kept 
"location" and "dimensions" in the "./my_types" folder and moved "product" into "./app_types", 
we could do this:

    var qb1 = require('qb1').init( 
        {prefix: 'catx', src: './my_types'}, 
        {prefix: 'zooboo', src: './app_types'} 
    )
    var product_type = qb1.type('zooboo/product')
    
To try a patch of the "location" type, we could put a new version of it into "./patch_types" and 
place it before "./my_types":

    var qb1 = require('qb1').init( 
        {prefix: 'catx', src: './patch_types'}, 
        {prefix: 'catx', src: './my_types'}, 
        {prefix: 'zooboo', src: './app_types'} 
    )
    var product_type = qb1.type('zooboo/product')

    
... and so on.