# openkv

LSM based key-value store in rust, design for cloud.

This project is in **Alpha** phase.
API and data layout will change rapidly.

# Goal

- [ ] Transactional key-value store:
    Provide transactional write, and snapshot read.

- [ ] Pluggable WAL: using local fs, raft log, or Kafka as a WAL provider.

- [ ] Minimize ser/de cost: 
    Access to serialized data in SSTable without deserializing.

- [ ] Design for large datasets:

    Reduce unnecessary IO with an efficient data index.
    Reduce in-memory data size by combining sparse-index and bloom filter.

- [ ] Flexible compaction policy:

    Reduces unnecessary compaction by comparing SSTable cardinality signature.
    Reduces IO consumption with partially SSTable merge down.
    Reclaims space quickly with cross-level SSTable merge.

- [ ] Design for the cloud:
    Stores less accessed SSTable on S3.

- [ ] Internal sharding
    A flattened structure reduces write/read amplification.
    A span with a heavy load will be pushed down more frequently than other spans.
