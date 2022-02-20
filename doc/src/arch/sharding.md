# Sharding

One of the performance issue about LSM tree is the write/read amplification.
When a db becomes bigger, the number of levels(`l`) in a LSM becomes increases, in
logarithm order.

A record will be rewritten([compaction][]) `l` times to enter the bottom level.
Assumes the fanout of every level is `n`,
every record amplifies write IO by `O(l) * n` times.

By splitting LSM into several smaller ones, `l` becomes smaller and the
write/read amplification will be reduced.

Thus openkv organize its data in a way resembles to a two-level BTree:
- The btree root node is a array of all sub-LSM, sorted in key order.
- Each of the leaf nodes is a small LSM.


```bob


                                                  .------------+----------+------------.
                                                  | "(-oo, b]" | "[b, e)" | "[e, +oo)" |
                                                  `------------+----------+------------'
                                                       |            |            '---------~~~
             .------.                                  |            `----------.
             | "L3" |                                  |                       |
             `------'                                  v                       v
             .------.                               .------.                .------.
             | "L2" |                               | "L2" |                | "L2" |
             `------'              ------>          `------'                `------'
       .------.    .------.                         .------.                .------.
       | "L1" |    | "L1" |                         | "L1" |                | "L1" |
       `------'    `------'                         `------'                `------'
 .------.    .------.    .------.             .------.    .------.    .------.    .------.
 | "L0" |    | "L0" |    | "L0" |             | "L0" |    | "L0" |    | "L0" |    | "L0" |
 `------'    `------'    `------'             `------'    `------'    `------'    `------'
```


## Sub-LSM

Sub-LSM is a small LSM tree, with a limited number of levels.



# Split and Merge

- A sub-LSM will be split if the level exceeds a threshold(e.g., 3).

- Two adjacent sub-LSM is merged into one if both of them becomes lower than 1/3 of the threshold.



[compaction]: compaction.md
