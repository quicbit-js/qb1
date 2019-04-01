# First Principles and Synergy

The qb1 format is founded on first principles and on experience working with a number of languages and
data storage systems.  The academically astute may find it disconcerting not to see a long list of 
cited articles and research pointing to reasons that back the decisions made in qb1.  But trying to create 
something that is as minimal in design as possible that blends strengths of type systems, data
expression, compression and encryption is more a matter of iterative trial and adjusting and
reducing than research.  To put it another way, accumulating research doesn't simplify the 
problem at hand.  The path to qb1 was an iterative and
almost obsessive fiddling and reduction of complexity and inefficiencies.  It is a path driven by
questions like:

* Why can't data be worked with simply, like JSON, yet powerfully like XML, yet efficiently like 
proprietary binary formats?

* Why do developers have do work with different tools and libraries to message data over the wire versus accumulate
it in files and perform simple fast scanning?

* What is the simplest and clearest way we can describe the overall structure of a data set?  Why can't 
that simple expression be also used to make information self-describing?  Why can't that same
structural definition be leveraged for better compression and to help align encryption with target
areas of information?

* Why can't developers have a format with the speed of binary but work with simplicity 
of text?

We use first principles to guide the construction of this qb1 format/serialization/storage/search:

## Principle: Smallest serialization unit is 8-bits

12-bit data and architecture faded away in the 70s and eking out single bit savings cramming a few 
5-bit words into 2 bytes is not very effective.  Our smallest unit of serialization is 8-bits.  
We  get greater space savings through recognizing structural patterns and redundancies than
by bit cramming.

## Principle: Format must be both human-readable and machine-efficient

To achieve a broad array of needs, qb1 data must support a human-readable canonical form that is 
in every way identical in function to compressed and binary forms.  We use JSON to work 
between human readable and binary complex forms and allow JSON to be substituted for 
almost any portion of binary data.  Metadata, types, subsets of values and/or all values
can be written in JSON equivalent to the binary forms.  Similarly, there are optional levels of 
compression and encryption that can be applied to qb1 files across select structures.

### Corollary: qb1 must include human-readable schema/structure information

In order to allow encoding specifications across different areas of a format, operators
must be able to view and describe those areas.  Therefore qb1 must have a clear and
well understood schema/structure on which to operate and set rules.

### Corollary: qb1 must include a selection / path semantics

In order to specify rules that apply to data structures, there must be a way to describe 
selections of data.  Therefore qb1 must have an means of specifying selections (e.g.
similar to xpath for XML).

## Principle: Data is self-described

Self-described data improves clarity, search, scanning and
data recovery.  The cost of self-described data can be made minimal
by using higher structural descriptions.  By allowing redundant structures to be described 
qb1 actually improves compression over conventional byte-level compression.

### Corollary: A description precedes every value

Every value in qb1 is preceded by a description of that value.  ***Note that this does not mean 
that every scalar value is preceded by a token***.  A vector (array) can be preceded by a description
of a vector and the possible type(s) of that vector.  For large vectors holding only a single type, 
this can considerably reduce storage overhead.

### Corollary: Complex types and patterns are describable

qb1 supports descriptions of complex types such as objects, arrays and any nesting complexity.  qb1
uses a concise syntax for describing data (this same corollary also falls from the human-readability principle)





