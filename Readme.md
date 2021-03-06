mongoid-js
==========

very very fast MongoID compatible unique id generator

Generates unique ids.  The ids are constructed like MongoDB document ids,
built out of a timestamp, system id, process id and sequence number.  Similar
to `BSON.ObjectID()`, but at 12 million ids / sec, 35 x faster.

The ids are guaranteed unique on any one server, and can be configured
to be unique across a cluster of up to 16 million (2^24) servers.
Uniqueness is guaranteed by unique {server, process} id pairs.


## Installation

        npm install mongoid-js
        npm test mongoid-js


## Summary

        var mongoid = require('mongoid-js');
        var id = mondoid();             // => 543f376340e2816497000001

        var MongoId = require('mongoid-js').MongoId;
        var idFactory = new MongoId(/*systemId:*/ 0x123);
        var id = idFactory.fetch();     // => 543f3789001230649f000001


## Functions

### mongoid( )

generates ids that are unique to this server.  The ids are generated by a
MongoId singleton initialized with a random machine id.  All subsequent calls
to mongoid() in this process will fetch ids from this singleton.

        // ids with a randomly chosen system id (here 0x40e281)
        var mongoid = require('mongoid-js').mongoid;
        var id1 = mongoid();            // => 543f376340e2816497000001
        var id2 = mongoid();            // => 543f376340e2816497000002

### MongoId( systemId )

unique id factory that embeds the given system id in each generated unique id.
By a systematic assignment of system ids to servers, this approach can guarantee
globally unique ids (ie, globally for an installation).

The systemId must be an integer between 0 and 16777215 (0xFFFFFF), inclusive.

        // ids with a unique system id
        var MongoId = require('mongoid-js').MongoId;
        var systemId = 4656;
        var idFactory = new MongoId(systemId);

#### id.fetch( )

generate and return the next id in the sequence.  Up to 16 million distinct
ids (16777216) can be fetched during the same wallclock second; trying to
fetch more throws an error.  The second starts when the clock reads _*000_
milliseconds, not when the first id is fetched.  The second ends 1000
milliseconds after the start, when the clock next reads _*000_ milliseconds.

        var id1 = idFactory().fetch();  // => 543f3789001230649f000001
        var id2 = idFactory().fetch();  // => 543f3789001230649f000002

#### id.parse( )

same as MongoId.parse(id.toString()), see below

#### id.getTimestamp( )

same as MongoId.getTimestamp(id.toString()), see below

#### id.toString( )

each MongoId object itself can have a distinct unique id, created on demand
when toString() is called.  The object invokes itself as an id factory and
fetches for itself the next id in the sequence.  The

### Class Methods

#### MongoId.parse( idString )

Decompose the id string into its parts -- unix timestamp, machine id,
process id and sequence number.  Unix timestamps are seconds since the
start of the epoch (1970-01-01 GMT).  Note that parse() returns seconds,
while getTimestamp() returns milliseconds.

        var parts = MongoId.parse("543f376340e2816497000013");
        // => { timestamp: 1413429091,
        //      machineid: 4252289,
        //      pid: 25751,
        //      sequence: 19 }

#### MongoId.getTimestamp( idString )

Return just the javascript timestamp part of the id.  Javascript timestamps
are milliseconds since the start of the epoch.  Each mongoid embeds a seconds
precision unix timestamp; getTimestamp() returns that multiplied by 1000.

        MongoId.getTimestamp("543f376340e2816497000013");
        // => 1413429091000


Change Log
----------

- 1.1.0 - tentative `browserify` support: use a random pid if process.pid is not set, avoid object methods in constructor, 100% unit test coverage
- 1.0.7 - fix getTimestamp and quantize correctly, deprecate index.js, test with qnit, fix sequence wrapping
- 1.0.6 - doc edits
- 1.0.5 - stable, fast version
