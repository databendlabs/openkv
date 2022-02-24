# Virtual SSTable

`VSSTable` is a unit to manage SSTable.
Because an SSTable may be referenced more than once:
e.g. when splitting a span.

```bob

       .~~~~~~.    .~~~~~~.                         .~~~~~~.        |        .~~~~~~.
       ! "V4" !    ! "V5" !                         ! "V4" !        |        ! "V5" !
       `~~+~~~'    `~~+~~~'                         `~~+~~~'        |        `~~+~~~'
 .~~~~~~. |  .~~~~~~. |  .~~~~~~.   split     .~~~~~~. |  .~~~~~~.  |  .~~~~~~. |  .~~~~~~.
 ! "V1" ! |  ! "V2" ! |  ! "V3" !  -------->  ! "V1" ! |  ! "V2" !  |  ! "V2'"! |  ! "V3" !
 `~~+~~~' |  `~~+~~~' |  `~~+~~~'             `~~+~~~' |  `~~~~~~'  |  `~~~~~~' |  `~~+~~~'
    |     |     |     |     |                    |     |      |     |     |     |     |
    |     |     |     |     |                    |     |      |     |     |     |     |
    |     |     |     |     |                    |     |      |     |     |     |     |
    |     |     |     |     |                    |     |      |     |     |     |     |
    |     |     |     |     |                    |     |      |     |     |     |     |
    |     v     |     v     |                    |     v      |     |     |     v     |
    |  .------. |  .------. |                    |  .------.  '---.   .---'  .------. |
    |  | "T4" | |  | "T5" | |                    |  | "T4" |      |   |      | "T5" | |
    v  `------' v  `------' v                    v  `------'      v   v      `------' v
 .------.    .------.    .------.             .------.           .------.          .------.
 | "T1" |    | "T2" |    | "T3" |             | "T1" |           | "T2" |          | "T3" |
 '------'    `------'    `------'             '------'           `------'          '------'
```

A SSTable is removed when there is no VSSTable referencing it.

A VSSTable has a key range that is subset of the SSTable it references.


```
struct VSSTable {
    key_start: String,
    key_end: String,
    sstable: SSTableId,
}
```
