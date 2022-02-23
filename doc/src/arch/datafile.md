# DataFile

Multiple SSTable are stored in a large file, the DataFile.

DataFile only allocates or reclaims space by a fixed size: its allocation unit.
There is one DataFile for each allocation unit.

Assuming in openkv the allocation unit are `4MB` and `16MB`, 
then there are two `DataFile`s: `df-4mb` and `df-16mb`.

Each DataFile has a corresponding bitmap for tracking allocated unit.

The bitmap is part of the [manifest][]

```bob

                  +-+-+-+-+-+
"bitmap-4mb:"     |1|0|1|1|0|
                  +++-+++++-+
                   |   | '--------------.
                   |   '---------.      |
                   v             v      v
                  +------+------+------+------+------+
"DataFile-4mb:"   |"SST0"|      |"SST0"|"SST0"|"..." |
                  +------+------+------+------+------+


                  +-+-+-+-+-+
"bitmap-16mb:"    |1|0|1|1|0|
                  +++-+++++-+
                   |   | '--------------.
                   |   '---------.      |
                   v             v      v

                   "..."

```

## Commit a SSTable

- Find the first `0` in df-bitmap.
    This bitmap has to be indexed to speed up searching for the first `0`.

- Flush and fsync SSTable data to DataFile.

- Flush and fsync bitmap in [manifest][].



[manifest]: manifest.md

