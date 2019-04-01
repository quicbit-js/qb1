# packets, series, and areas

Consider the following data organization/design:

big -> small

    area     - a master namespace consisting of one or more data series (definitions and data)
    series   - an ordered collection of packets sharing a common structure or form (which may evolve)
    packet   - a unit of delivered data (compressable/encryptable in whole or part) consisting of one or more chunks
    chunk    - a self-described segment of data

* area (of series)
    * header (area definition)
    * stats
    * index
    * series headers (collection of just header information)
    * series
    * data definitions
      * rules for how data is defined and produced in cluding partition rules
    

* series (of packets)
    * header  (series definition including partition rules)
    * stats
    * index
    * packet headers
    * packets
    
* packet (of chunks)
    * header
    * stats
    * index
    * chunk headers
    * chunks - compressable/encryptable in whole or part
     
* chunk (of segments)
    * header
    * stats 
    * index
    * segment headers
    * segments - independently compressable and encryptable
    

Notice the strong similarities, especially the attributes of series, packet and chunk - 
almost identical.  Perhaps these should all be the same thing and instead of 
having fixed levels of organization, we have simply:

* packet (of packets)
    * header
    * stats
    * index
    * child packet headers
    * child packets 

The advantage of this is that packets can be organized an any needed level.  Depending on 
data sizes and serialization requriements, we might put hundreds of packets in a file
or split the packet in to child packets, or create a parent packet to encompass
a large number of packets.  Every time we add a level we gain control and visibility
over a large set of data.  The control spans data logarithmically and can keep up
with any data growth (which also tends to be exponential).  The design matches the
exponential growth challenges we so often encounter in businesses today.

Predefined Packets (series):

    type -          a data series that represents data types, constraints and forms including support for version and change.
    type/type -     special type defining the structure of type definitions themselves
    type/packet - defines packet structure
    type/series - structure defining how a particular data type is to be partitioned and sequenced


    
# Example Packet

    var qb1 = require('qb1')
    var rows = []
    for (var i=1; i<1000; i++) { rows.push({a: i, b: 's' + i}) }
    var packet = qb1.packet(rows)
    
    console.log( packet.json() )
    
    {
        type: [ {a:'int', b:'str'} ],  
        stats: {
            len: 999,
            a: { min: 1, max: 999,  },
            b: { min: 's1', max: 's999', len: "2..4" }
        },
        data: [
                { 
                    id: 1, 
                    bytes: 2612, 
                    csum: 'OU26nO'
                    index: [
                        { o: 0, bytes: 35, csum: '2nsodi2', ... },
                        { o: 36, bytes: 263, csum: 'honow2', ... },
                    ]
                },     
                [
                    "A3RognE=S...",     segment...
                    "ZOIHSO82...",      segment...
                ],
                ...
            },
            
            
        ]
    }
   