
- At runtime a linked list of stream source & intermediate operations is build & optimized, one per "stage" in pipline

강의자료를 보면 Spliterator는 각 input source 에서 구현되어 있는 듯 함

Source stage stream flags are derived from spliterator characteristics.

SIZED: size of stream is known
DISTINCT: elements of stream are distict (ex. HashSet, TreeSet)
SORTED: elements of the stream are sorted in natural order (ex. treeSet)
ORDERED: Stream has meaningful encounter order  (ex. treeSet)