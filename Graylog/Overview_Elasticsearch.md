# Configuring and tuning Elasticsearch
- Chúng tôi khuyên bạn nên sử dụng 1 Elasticsearch cluster chuyên biệt cho graylog của bạn.
- Nếu bạn sử dụng 1 Elasticsearch dùng chung, các vấn đề bạn với các index không liên quan đến graylog sẽ chuyển trạng thái graylog
thành màu vàng hoặc đỏ và làm ảnh hưởng đến tính khả dụng cũng như hiệu năng của graylog.

## Configuration
### Configuration của graylog-server nodes
- Điều quan trọng nhất là tạo kết nối thành công giữa Elasticsearch cluster và discovery mode.
- Graylog sử dụng multicast để có thể discover Elasticsearch nodes.

### Cluster name
- Bạn phải thông báo cho graylog-server biết Elasticsearch cluster nào sẽ join vào.
- Tên Elasticsearch cluster mặc định là "elasticsearch" và được cấu hình cho tất cả Elasticsearch node trong file cấu hình "elasticsearch.yml" với cú pháp "cluster.name".
- Cấu hình tương tự trong file "graylog.conf" với cú pháp "elasticsearch_cluster_name"
- Chúng tôi khuyên bạn nên đổi nó thành graylog-production hoặc là gì theo ý bạn chứ không nên để mặc định.
- file "elasticsearch.yml" nằm trong đường dẫn /etc/elasticsearch/.

### Discovery mode
- Chế độ discovery mặc định là multicast.
- Graylog sẽ cố gắng tìm những Elasticsearch nodes khác 1 cách tự động.
- Điều trên sẽ hoạt động tốt trong hệ thống nhỏ, thế nhưng sẽ gặp vấn đề ngay lập tức nếu chạy trong topo mạng lớn.
- Chúng tôi khuyên bạn nên sử dụng unicast.
- Cấu hình Zen unicast discovery trong Graylog với các dòng sau ở trong file configuration của bạn.
```sh
# Disable multicast
elasticsearch_discovery_zen_ping_multicast_enabled = false
# List of Elasticsearch nodes to connect to
elasticsearch_discovery_zen_ping_unicast_hosts = es-node-1.example.org:9300,es-node-2.example.org:9300
```

- Bạn cũng phải đồng thời cấu hình Zen unicast discovery trong file cấu hình Elasticsearch 
```sh
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["es-node-1.example.org:9300" , "es-node-2.example.org:9300"]
```

- Port giao tiếp mặc định của Elasticsearch là 9300/tcp
- Port có thể được thay đổi trong file conf Elasticsearch với cú pháp "transport.tcp.port".

## Configuration of Elasticsearch nodes
### Disable dynamic scripting
- Elasticsearch có 1 cấu hình mặc định không an toàn đó là cho phép execute code từ xa (see <a class="reference external" href="http://bouk.co/blog/elasticsearch-rce/">here</a> and <a class="reference external" href="https://groups.google.com/forum/#!msg/graylog2/-icrS0rIA-Q/cCTJaNjVrQAJ">here</a> for details)</p>
- Bạn hãy chắc chawscn add cú pháp sau "script.disable_dynamic: true" vào trong file "elasticsearch.yml" để disable dynamic scripting.

### Control access to Elasticsearch ports
- Elasticsearch không có cơ chế xác thực nào, nên bạn phải hạn chế truy cập tới Elasticsearch ports (mặc định: 9200/tcp và 9300/tcp).
- Nếu không thì bất kì ai có kết nối tới Elasticsearch có thể đọc đươc dữ liệu.

### Open file limits
- Bởi vì Elasticsearch phải giữ rất nhiều files mở đồng thời nó cũng yêu cầu giới hạn file mở nhiều hơn so với sự cho phép.Set cho nó ít nhất 64000 file mở.
- Graylog sẽ show thông báo trên web interface khi có 1 node trong Elasticsearch cluster giới hạn mở file quá thấp.

### Heap size
- Lời khuyên là bạn nên tăng size tiêu chuẩn của bộ nhớ heap được chỉ định cho Elasticsearch.
- VD:Hãy set biến môi trường "ES_HEAP_SIZE" 24g.
- Lời khuyên nữa là bạn nên dùng khoảng 50% bộ nhớ hệ thống khả dụng cho Elasticsearch (Khi chạy trên 1 host riêng biệt) để tạo ra đủ space cho system caches mà Elasticsearch sử dụng.

### Merge throttling
- Elasticsearch bóp nghẹt sự kết hợp các Lucene segments để có thể tìm kiếm nhanh ngay lập tức.
- Tuy nhiên điều này cũng có thể làm giảm tốc độ của Graylog.
- Khi bạn SSD hoặc SAN , bạn có thể tăng giá trị của "indices.store.throttle.max_bytes_per_sec" lên 150MB

## Cluster Status explained
### RED
- Trạng thái màu đỏ chỉ ra rằng 1 vài primary shards không khả dụng.
- Trong trường hợp này thì không thể perfom bất kì tìm kiếm nào cho đến khi primary shards được khôi phục.

### YELLOW
- Trạng thái màu vàng nghĩa là tất cả primary shards khả dụng nhưng có 1 vài shards replica không ổn.

### GREEN
- 	Tất cả đều ổn
