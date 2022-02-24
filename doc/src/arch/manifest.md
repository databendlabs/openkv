# Manifest

Manifest in openkv itself is a tiny db with WAL of operations to the system and a snapshot.


Snapshot:

```
version

data_file_bitmaps: {
    "4mb":  Bitmap
    "16mb": Bitmap
}

spans: [
{
    start: String
    end: String
    vsst: {
        0: [VS1, VS2, ...]
        1: [VSi, VSj, ...], 

    }

}

]
    



```
