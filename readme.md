# qb1

    The speed of binary,
        the security of encryption,
            the ease of text,
                and the tools to manage it all with confidence.


## Overview

A new data distribution and storage technology that simplifies and improves management of data sets.  
The technology improves reliability, speed and reduces costs of storage and processing large volumes of data.

qb1 is actually three technologies in one:

1) an intuitive and powerful modeling language
2) model-aligned compression
3) self-described serialization-and-storage

The combination of these technologies is far more powerful than separate offerings because of the synergies that are achieved by all three working together.

## Modeling

qb1 provides a simple and powerful modeling language.  It is designed to be both intuitive and compact while still being human readable.  We will get to some examples below.

Model-Aligned Compression

Unlike common byte-level compression such as zip or snappy, qb1 compression retains the macro structures found in data files like CSV or JSON.  By honoring the structure of data during compression qb1 achieves:

1) better compression ratios of structured information up to 10 or even 100's of times better than zip.
2) faster search through selective and fast forward skipping without expensive decompression
3) synergy with qb1 serialization brings together packet format with file formats to allow one-time compression for files over the wire and on disk.

Self-Described Serialization and Storage

All qb1 data sent over the wire and stored to disk is self-described using qb1 intuitive modeling.  Modeled data can be read and used by programs without any setup or configuration.  qb1 is instantly and automatically accessible and usable.  The format is designed to be simple enough that it can be used years of even decades later.  The combination of simplicity and efficiency makes it a great choice for data distribution and storage.
Designing a universal format
No format has achieved universal domination, but we think qb1 can.  One of the most impressive format changes examples is JSON.  It has had lightning-pace adoption across all languages because of its simplicity and relative efficiency over XML, but JSON includes no modeling concept (multiple competing specs), fails for large documents, and is inefficient when compared with data formats like Apache Parquet.  But efficient formats like Parquet will never be universal because they aren't designed to be general serialization formats.  Formats like BSON (Mongo DB's binary JSON format) have biases that prevent universal adoption.
qb1 is designed to become the universal serialization and storage format for three reasons:

1) Simplicity
2) Depth
3) Speed

* Simplicity - qb1 is expressible in JSON so humans can read and understand it.  It has direct translation to and from the most popular format.  This also makes it very adoptable because systems can move from JSON to qb1 and back again.
* Depth - the underlying qb1 format uses special 'chain' encoding to support any-size values.  There is no machine bias.  64 bit floats, 128 bit floats, 1024 bit floats... there is no limit because we store the data using integer-chain technology.  Any size value, any precision, any size string.
* Speed - qb1 is extremely fast to transfer, store and read.  The compactness and serialization features make it very immediately appealing.
Example of Modeling and Self-Describing Data

Data that is self-described over the wire and on disk must include a model of the data with the data itself.  qb1 uses a concise and powerful modeling language to achieve more compact, yet still simple self-described data.  

To illustrate, here is an example of json-schema, a popular modeling format:

    {
      "type": "array",
      "items": {
        "title": "product",
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
            }
          },
          "dimensions": {
            "type": "object",
            "properties": {
              "length": {"type": "number"},
              "width": {"type": "number"},
              "height": {"type": "number"}
            }
          }
        }
      }
    }
    
Here's that same model using qb1:

    [
      {
        "$name": "product",
        "id": "integer",
        "name": "string",
        "price": "0.0~~",
        "tags": [ "string" ],
        "dimensions": {
          "length": "0.0..",
          "width": "0.0..",
          "height": "0.0.."
        }
      }
    ]
    
    ("0.0~~" is a range specification meaning a decimal from 0.0 to infinity, excluding 0 and "0.0.." means the range from 0.0 to infinity, including zero.)

Here's the model again using the qb1 most concise format with single-character types for common types ('i' for integer, 's' for string):

    [{"$name":"product","id":"i","name":"s","price":"0.0~~","tags":["s"],"dimensions":{"length":"0.0..","width":"0.0..","height":"0.0.."}}]

The model is so compact that even as JSON text it rivals the compactness of binary (which qb1 also supports).  This means a self-describing model data can be written on every data packet with minimal cost.  No other serialization today incorporates this modeling clarity and conciseness with compression.  Again, the power of self-describing compressed data is multi-fold:
better compression, improved reliability, faster seek times, reduced CPU costs.  Many benefits.

## Further Reading

* [principles upon which qb1 is designed](https://github.com/quicbit-js/qb1/blob/master/ref/first-principles.md)
* [foundations of qb1 binary encoding](https://github.com/quicbit-js/qb1/blob/master/ref/qb1-encoding-foundation.md)
* [qb1 range syntax](https://github.com/quicbit-js/qb1/blob/master/ref/ranges.md)
* [complex type representation](https://github.com/quicbit-js/qb1/blob/master/ref/complex-types.md)
* [comparison with json-schema](https://github.com/quicbit-js/qb1/blob/master/ref/json-schema.md)
