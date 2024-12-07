
### Trees fo Disk-Based Storage

to maintain a BST on disk, several problem exists.

- Locality
	- elements are added in random order. no guarantee that a newly created node is written close to its parent. child may span across several disk pages. -> paged binary tree 로 개선 가능
- Tree height
	- 