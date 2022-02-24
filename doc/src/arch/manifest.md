# Manifest

Manifest in openkv itself is a tiny db with WAL of operations to the system and a snapshot.


Snapshot:

```
version: String

data_file_bitmaps: {
    "4mb":  Bitmap
    "16mb": Bitmap
}

spans: [
    {
        start: String
        end: String
        vsstables: {
            0: [VSST1, VSST2, ...]
            1: [VSSTi, VSSTj, ...],
        }
    },
    ...
]

separated_values: [
    {
        start_value_id: String,
        end_value_id: String
        vsstables: {
            0: [VSST1, VSST2, ...]
            1: [VSSTi, VSSTj, ...],
        }

    },
    ...

]

```
