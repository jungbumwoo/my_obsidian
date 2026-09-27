
pointer를 줄여서 메모리 사용량을 효율화함.

1. Prevlen: This field stores the length of the previous entry. It allows for efficient traversal in both forward and backward directions within the ziplist.
2. Entrylen: This field stores the length of the current entry.
3. Content: This field contains the actual data of the element, such as a string, integer, or floating-point value


Redis automatically switches between ziplist and other data structures, such as linked lists or hash tables, based on **the number of elements and their sizes.**

