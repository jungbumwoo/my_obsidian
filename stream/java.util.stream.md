
- No storage. it conveys elements from a source such as a data structure, an array, a generator function, or an I/O channel, through a pipeline of computational operations.
- Functional in nature. An operation on a stream produces a result, but does not modify its source. 
- Laziness-seeking. Many stream operations, such as filtering, mapping, or duplicate removal, can be implemented lazily. For example, "find the first `String` with three consecutive vowels" need not examine all the input strings. Stream operations are divided into intermediate (`Stream`-producing) operations and terminal (value- or side-effect-producing) operations. Intermediate operations are always lazy.
- Possibly unbounded.  can allow computations on infinite streams to complete in finite time.
- **Consumable.** The elements of a stream are only visited once during the life of a stream. Like an [`Iterator`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Iterator.html "interface in java.util"), a new stream must be generated to revisit the same elements of the source.

---
https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html

