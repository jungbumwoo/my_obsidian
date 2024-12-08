
### Trees fo Disk-Based Storage

to maintain a BST on disk, several problem exists.

- Locality
	- elements are added in random order. no guarantee that a newly created node is written close to its parent. child may span across several disk pages. -> paged binary tree 로 개선 가능
- Tree height
	- 2-3 Trees and other low-fanout trees have a similar limitation. usefual as in-memory data structures, but small node size makes them impractial for external storage.

### Disk-Based Structures

HDDs and SSDs are addressing chunks of memory rather than individual bytes, most operating systems have a block device abstraction. When we're reading a single word from a block device, the whole block containing it is read.

In SSDs, we don't have a strong emphasis on random vs sequential I/O