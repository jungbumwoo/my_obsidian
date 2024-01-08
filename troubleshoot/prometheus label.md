
prometheus label 이 너무 많이 생성되어서 프로메테우스가 터짐

`./data/wal` 내부 파일들 삭제 후 복구


`wal` compacted 처리 되기 전 raw_data 들이 보관 되는 곳.
server start 할 때 replay되는데 이게 너무 커져서 prometheus server가 다시 뜨지도 못함

https://prometheus.io/docs/prometheus/latest/storage/