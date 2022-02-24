# Min-hash

[min-hash][] is a signature of all keys in a SSTable and is used to optimize the
compaction. E.g., find out a pair of SSTable with most common keys.

# Jaccard similarity

Let `U` be a set and `A` and `B` be subsets of `U`,
then the **Jaccard index** is defined to be the ratio of the number of elements of their intersection and
the number of elements of their union:

$$
J(A,B)={{|A\cap B|} \over {|A\cup B|}}.
$$


Let `h` be a hash function that maps the members of `U` to distinct integers, e.g., let `h(x) = SHA1(x)`.

For any set `S`, define `H_min(S)` to be the minimal integer(hash value) of members in `S`:

H_min(S) = min({h(x) | x ∈ S})

Now, applying `H_min` to both `A` and `B`,
and assuming no hash collisions,
the probability that `H_min(A) == H_min(B)` is true is equal to the similarity `J(A,B)`:

Pr[ H_min(A) = H_min(B) ] = J(A,B)

# Estimate the number of common keys

Given two SSTable, the number of common keys can be calculated with:

$$
c = \frac{p}{1+p} (|A| + |B|)
$$


# Estimate the Probability

The probability can be estimiated with `y/k`,
where `k` is the number of different hash functions
and `y` is the number of hash functions hᵢ for which hᵢ(A) == hᵢ(B)


### Generate signature for a SSTable

```rust
fn gen_sstable_signature(sstable: &SSTable, number_of_hashes: usize) -> Vec<u64>{

    // Result signature
    let signature = Vec::with_capacity(number_of_hashes);

    // Distribute hash values to `k` buckets to simulate `k` hash functions.
    let buckets = Vec::with_capacity(number_of_hashes);

    for k in sstable.keys() {
        let h = SHA1(k) as u64;
        buckets[h%number_of_hashes].push(h);
    }

    for i in 0..number_of_hashes {
        signature[i] = min(buckets[i]);
    }

    signature
}
```

### Calculate similarity of two SSTable


```rust

fn calc_sig(a: &SSTable, b:&SSTable) -> f64 {
    let eq = 0.0;
    for i in 0..number_of_hashes:
        if a.signature[i] == a.signature[i] {
            eq += 1
        }
    eq / number_of_hashes

}
```

## Simulation

We provides a python script [min-hash.py][] to estimate the accuracy of this algo.

| Config | value |
| :--    | --: |
| Number of hash functions(buckets) |  128 |
| Hash value | u64 |
| Space cost | sizeof(u64) * 128 = 1KB |

Actual vs Estimated:

| \|A∪B\| | \|A\|   | \|B\|   | Actual (A∩B)/(A∪B)% |  Estimated% | error% |
| --:     | --:     | --:     | --:                 |  --:        | --:    |
| 1000    | 360     | 840     | 20.00%              |  21.88%     | 1.87%  |
| 1000    | 520     | 880     | 40.00%              |  38.28%     | -1.72% |
| 1000    | 680     | 920     | 60.00%              |  60.94%     | 0.94%  |
| 1000    | 839     | 959     | 80.16%              |  78.91%     | -1.25% |
| 1000    | 1000    | 1000    | 100.00%             |  100.00%    | 0.00%  |
| 10000   | 3600    | 8400    | 20.00%              |  15.62%     | -4.38% |
| 10000   | 5200    | 8800    | 40.00%              |  35.16%     | -4.84% |
| 10000   | 6800    | 9200    | 60.00%              |  60.94%     | 0.94%  |
| 10000   | 8399    | 9599    | 80.02%              |  85.16%     | 5.14%  |
| 10000   | 10000   | 10000   | 100.00%             |  100.00%    | 0.00%  |
| 100000  | 36000   | 84000   | 20.00%              |  21.88%     | 1.87%  |
| 100000  | 52000   | 88000   | 40.00%              |  47.66%     | 7.66%  |
| 100000  | 68000   | 92000   | 60.00%              |  62.50%     | 2.50%  |
| 100000  | 83999   | 95999   | 80.00%              |  80.47%     | 0.47%  |
| 100000  | 100000  | 100000  | 100.00%             |  100.00%    | 0.00%  |
| 1000000 | 360000  | 840000  | 20.00%              |  19.53%     | -0.47% |
| 1000000 | 520000  | 880000  | 40.00%              |  40.62%     | 0.62%  |
| 1000000 | 680000  | 920000  | 60.00%              |  58.59%     | -1.41% |
| 1000000 | 839999  | 959999  | 80.00%              |  75.78%     | -4.22% |
| 1000000 | 1000000 | 1000000 | 100.00%             |  100.00%    | 0.00%  |


# Optimize compaction with min-hash

With min-hash we can find the best choice to compact.

For every SSTable,
calculate the Jaccard index of it and the overlapping SSTable at the lower level.

Push down the one with the max Jaccard index.

- The space cost is negligible. Only `1KB` more space for every SSTable.

- The time complexity is `O(n)`, where `n` is the number of SSTable in the
    system. Because for every SSTable, there are about only `k` SSTable that
    have overlapping key range with it.where `k` is the level fanout.


[min-hash]: https://en.wikipedia.org/wiki/MinHash
[min-hash.py]: https://drmingdrmer.github.io/post-res/compact/min-hash.py
