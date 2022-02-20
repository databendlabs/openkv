# SSTable

A SSTable(solid-state table) is the same concept used in levelDB or rocksDB.

The size of an SSTable is `16MB`.

The data layout inside an SSTable:


```bob
.-----------+---------+---------+-------------+-----------------------.
| "Header"  | "Metas" | "Index" | "Records"   | "Checksum"            |
`-----------+---------+---------+-------------+-----------------------'
```

## Header

`Header` is fix-sized and contains an SSTable format version etc.

## Metas

`Metas` stores configs of this SSTable, such as key types, value types, etc.
E.g., a fix-sized key or value may use a type-specific store format.

Fix-sized key does not need to store key-suffix in `Records` segment.
TODO: prove it.

`Metas` also stores the cardinality signature of all keys to optimize SSTable compaction.


## Index

`Index` is a trie built from all keys stored in this SSTable, without
single-branch tail.
The leaf node in the trie stored the corresponding offset of a record in
`Records`.

It contains enough info to locate a present key in the `Records` segment.
But there could be a false-positive for a key not in `Records`.

For a given example of 3 key values:

```
fantastice: 1
foo: 5
food: 3
```

The Index would be like this:

```bob

.-.      .-.
|f|-+--->|a| : "offset for fantastice"
'-' |    '-'
    |
    |    .-.    .-.
    `--->|o|--->|o| : "offset of foo"
         '-'    '-'
                 |
                 |     .-.
                 `-->  |d| : "offset of food"
                       '-'

```

### Caching

`Index` is preferred to reside in memory, while the `Records` is left on disk.

### Sparse Index

Not all keys in the `Index` has a corresponding offset stored for it.
Since the IO optimization should be IO-oriented, reading several bytes may cost
almost the same resource as reading several kilo-bytes.
Thus the `Index` only need to store an offset for several adjacent keys.

And we just need to limit the distance between two adjacent offsets in the
`Index` to be less than a expected value, e.g., 16 KB.




## Records

The `Records` segment stores key suffixes that are not included in `Index` and
value.

`Records` is indexed by offsets that are stored in `Index`.

`key-suffix` may be empty
`value` is var-len serialized `bytes`

```bob

.--------------+---------.
| "key-suffix" | "value" |
|--------------+---------|
| "key-suffix" | "value" |
|--------------+---------|
| "..."        | "..."   |
'--------------+---------'


```



## Checksum

SHA256 of all preceding bytes in the SSTable.


# Reading a record

- Search the key in the `Index`. If it matches a path in the trie,
    scan `Records` from the offset upto next offset to find the record.

- If the key-suffix also matches the search key, returns the value.


# Performance considerations

- Trie naturally compresses prefixes and an SSTable only stores a small range of keys
  thus the space cost is low enough.

  For a static trie, succinct data structure can be adopted to reduce the size
  of the trie.

  Our goal budget for a key is about several bytes. So that keeping the `Index`
  in memory is possible and unnecessary IO can be reduced.

- Trie is not cache friendly thus a single trie should not be very large.
    The expected time cost for a query in the `Index` is less than 200 ns.
    
