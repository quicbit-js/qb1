qb has notation for all inclusive and exclusive range combinations and the empty-range:

    closed-closed:  "0..10"     (includes zero and 10)
    closed-open:    "0..~10"    (includes zero, but excludes 10)
    open-closed:    "0~..10"    (excludes zero, but includes 10)
    open-open:      "0~~10"     (excludes zero and 10)
    empty:          "<>"        (empty range)
    
Leaving out the low or high limit will default the values to 
Number.NEGATIVE_INFINITY and Number.POSITIVE_INFINITY, respectively.  Excluding infinity will disallow 
this special value.

    0..             zero-or-above, allowing infinity
    0..~            zero-or-above, excluding infinity
    0~..            above zero, allowing infinity
    0~~             above zero, excluding infinity
    ..~10e6         below one million, allowing -infinity
    ~~10e6          below one million, excluding -infinity
    ~~              any number other than (+/-) infinity
    ..              any number including (+/-) infinity
    ..~             any number including -infinity, excluding infinity
    
    etc...
   
    
A single number in place of a range is interpreted as a single value range:

    1               is equivalent to        1..1
    -3.7e-17        is equivalent to        -3.7e-17..-3.7e-17
    
Range is always read left-to-right as low-to-high and a range with no content is interpreted as the empty set:

    1..-1           is equivalent to        <>
    5..1            is equivalent to        <>
    
    etc...
    
    

## Range Types

Numeric type will be inferred from a range strings based on fraction parts.  If any part is 
a fraction, the whole interval will be interpreted as a decimal type:

    1..10           is equivalent to        int(1..10)
    1.0..10.0       is equivalent to        dec(1..10)
    
This shorter form is the canonical form for ranges, so 

    int(1..10)      has canonical form      1..10
    dec(1..10)      has canonical form      1.0..10.0
    
Float ranges (binary floating point) when written as strings must use the float type 
specifier ('float', or 'flt', or 'f') to 
be interpreted as floating point binary.  *float() ranges use hexidecimal strings 
to decimal notation to represent values*.

    flt(F2.A12)
    
By correctly applying floating point and decimal, qb retains a 100% lossless and faithful account 
of scientific and banking numbers.


## Range Sets and Enumerations

In most places where a range can be specified, multiple ranges are accepted:

    
    int(~~0,0~~)            any integer except zero or +/- infininty
    ~~0,0~~                 same as int(~~0,0~~)
    
    1..2,4..7,9..12

Since single values are interpreted as single value inclusive ranges, individual values are also acceptable:

    1,2,4,5,6,7,9,10,11,12
    
For numbers, you can use number lists in place of of the enum keyword:

    int(2,4,6,8,10)     is equivalent to                    int(enum(2,4,6,8,10))
    
Since both ranges and enums interpret type, the 'int' specifier isn't required in either case:

    2,4,6,8,10          is equivalent to the above and      enum(2,4,6,8,10)
    
Similarly, an enum can include *finite* ranges which are equivalent to enumerated sets:

    enum(5..10,13)         is equivalent to        enum(5,6,7,8,9,10,13)   
            
