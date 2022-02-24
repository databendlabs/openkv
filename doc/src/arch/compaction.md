# Compaction

Push a SSTable down and merge it with all overlapping SSTable at the next lower
level.

Leave only the record with the greatest `seq` when merging.

When reaching the bottom level, i.e, level-0, a tombstone record can be removed
for good.
