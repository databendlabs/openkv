# MemTable

MemTable in LSM is the in-memory representation of key-value pairs.

It can be considered as a `BTreeMap`.


There is only one active `MemTable` for every openkv instance.
Memtable is shared by all [sub-LSM][]


# Flush MemTable to SSTable

If the MemTable becomes too large, flush one continous portion(span) into a
sub-LSM.

MemTable always flushes the biggest span, e.g., the one with most keys, unless
there is a pressure to reclaim WAL space.


[sub-LSM]: sharding.md#sub-lsm
