# How To Install Graylog 1.x on CentOS 7
## Mục lục
[1. Giới thiệu] (#1)

[2.Cài đặt MongoDB] (#2)

[3.Cài đặt Java] (#3)

[4.Cài đặt Elasticsearch] (#4)

[5.Cài đặt Graylog-Server] (#5)

[6.Cài đặt Graylog-Web] (#6)

[7.Cấu hình Rsyslog gửi syslogs tới Graylog-Server] (#7)

[8.Tham Khảo] (#8)

<a name="1"></a>
### 1.Giới thiệu
- Trong bài này chúng ta sẽ tìm hiểu cách cài đặt Graylog v1.3.x (hay còn được gọi là Graylog2) trên CentOS 7, và cấu hình để tập trung các syslogs của hệ thống tại 1 nơi trung tâm.
- Graylog là 1 tool quản lý và phân tích log rất mạnh mẽ được sử dựng trong rất nhiều trường hợp, từ giám sát đăng nhập ssh đến các hoạt động không thường xuyên để có thể debug app.
- Nó dựa trên trên Elasticsearch, Java, MongoDB.
- Graylog có 4 thành phần chính:
<ul>
	<li>Node Graylog Server: có nhiệm vụ nhận và xử lý bản tin,kết nối tới tất cả các thành phần còn lại.Hiệu năng của nó phụ thuộc vào CPU.</li>
	<li>Node Elasticsearch: lưu trữ toàn bộ logs,bản tin, và giúp quản trị truy vấn và tìm kiếm logs.Hiệu năng của nó phụ thuộc vào Ram.</li>
	<li>MongoDB: Lưu trữ dữ liệu lớn.</li>
	<li>Web Interface: giao diện người dùng.</li>
</ul>
- Lược đồ về các thành phần graylog ( chú ý rằng messages được gửi từ server #) 
<img src="http://i.imgur.com/Fat8psW.png /">
*Chú ý: bạn phải đăng nhập với tài khoản root và server của bạn phải có ít nhất 2Gb Ram.*
- Tiến hành cài đặt nào ^_^

<a name="2"></a>
### 2.Cài đặt MongoDB
- Chạy câu lệnh sau để import MongoDB PGP key vào rpm:
```sh
rpm --import https://www.mongodb.org/static/pgp/server-3.2.asc
```
- Tạo MongoDB source list:
```sh
echo '[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
gpgcheck=1
enabled=1' | sudo tee /etc/yum.repos.d/mongodb-org-3.2.repo
```
- Tiến hành cài đặt:
```sh
yum install -y mongodb-org
```
- Nếu bạn sử dụng SElinux, bạn phải cài đặt packages sau để cấu hình SElinux
```sh
yum -y install policycoreutils-python
```
- Chạy câu lệnh sau để SElinux cho phép MongoDB có thể khởi chạy.
```sh
semanage port -a -t mongod_port_t -p tcp 27017
```
- Khởi chạy dịch vụ
```sh
systemctl restart mongod
chkconfig mongod on
```

<a name="3"></a>
### 3.Cài đặt Java
- Elasticsearch và Logstash yêu cầu phải có Java.
- Di chuyển tới thư mục `home` của bạn và download Oracle Java 8 JDK RPM:
```sh
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u73-b02/jdk-8u73-linux-x64.rpm"
```
- Cài đặt
```sh
yum -y localinstall jdk-8u73-linux-x64.rpm
```
- Bạn có thể xóa file vừa download:
```sh
rm ~/jdk-8u*-linux-x64.rpm
```

<a name="4"></a>
### 4.Cài đặt Elasticsearch
- Graylog 1.x chỉ có thể tương thích với Elasticsearch version < 2.0, nên ta sẽ cài đặt Elasticsearch 1.7.x.
- Chạy câu lệnh sau để import Elasticsearch PGP key vào rpm:
```sh
rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch
```
- Tạo file yum repo mới cho Elasticsearch :
```sh
echo '[elasticsearch-1.7]
name=Elasticsearch repository for 1.7.x packages
baseurl=http://packages.elastic.co/elasticsearch/1.7/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1' | sudo tee /etc/yum.repos.d/elasticsearch.repo
```
- Cài đặt
```sh
yum -y install elasticsearch
```
- Tiến hành cấu hình:
```sh
vi /etc/elasticsearch/elasticsearch.yml
```
- Tìm đoạn text `cluster.name`.Uncomment và thay giá trị mặc định với `graylog2` (tên này bạn tùy chọn).
```sh
cluster.name: graylog2
```
- Bạn sẽ muốn hạn chế các truy cập bên ngoài vào Elasticsearch (port 9200), để người bên ngoài không thể đọc dữ liệu của bạn hoặc shutdown Elasticsearch của bạn thông qua HTTP API. tìm đoạn text `network.host`, uncomment và thay giá trị mặc định với "localhost".
```sh
network.host: localhost
```
- Nếu bạn chỉ có 1 Elasticsearch trên server này.Thay đổi
```sh
index.number_of_shards: 1
```
- Uncomment và chỉnh sửa 2 dòng sau:
```sh
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["YourGrayLogIP:9300"]
```
- Thêm 1 dòng vào cuối file để disable dynamic script tránh cho việc execution từ xa.
```sh
script.disable_dynamic: true
```
- Lưu lại và chạy Elasticsearch
```sh
systemctl restart elasticsearch
```
- Khởi chạy Elasticsearch khi khởi động server
```sh
systemctl enable elasticsearch
```
- Sau 1 vài giây, test Elasticsearch có đang chạy đúng không:
```sh
curl -XGET 'http://localhost:9200/_cluster/health?pretty=true'
```
<img src="http://image.prntscr.com/image/b8cae51293e9406981ae88af922003fc.png" />
- Nếu status là green thì là ok.
<a name ="5"></a>
### 5.Cài đặt Graylog-Server
- Đầu tiên download Graylog RPM package :
```sh
rpm -Uvh https://packages.graylog2.org/repo/packages/graylog-1.3-repository-el7_latest.rpm
```
- Cài đặt
```sh
yum -y install graylog-server
```
- Cài đặt pwgen để tạo password secret keys:
```sh
sudo yum -y install epel-release
sudo yum -y install pwgen
```
- Chúng ta tạo ra 1 random key và chèn nó file vào file configure.
```sh
SECRET=$(pwgen -s 96 1)
sed -i -e 's/password_secret =.*/password_secret = '$SECRET'/' /etc/graylog/server/server.conf
```
- Tiếp theo đến tạo password cho admin
```sh
PASSWORD=$(echo -n *YourPassword* | sha256sum | awk '{print $1}')
sed -i -e 's/root_password_sha2 =.*/root_password_sha2 = '$PASSWORD'/' /etc/graylog/server/server.conf
```
- Cấu hình file Configure
```sh
vi /etc/graylog/server/server.conf
```
- Chỉnh sửa các tham số sau:
```sh
root_timezone = Asia/Ho_Chi_Minh
rest_listen_uri = http://0.0.0.0:12900/
rest_transport_uri = http://0.0.0.0:12900/
elasticsearch_shards = 1
elasticsearch_cluster_name = graylog2 (1)
elasticsearch_http_enabled = false
elasticsearch_discovery_zen_ping_multicast_enabled = false
elasticsearch_discovery_zen_ping_unicast_hosts = YourGrayLogIP:9300
mongodb_useauth = false
```
- (1)Chú ý: tên này phải giống tên cluster đã đặt trong file elasticsearch.yml.
- Khởi chạy
```sh
systemctl start graylog-server
```
- Bạn có thể kiểm tra server khởi động, giúp bạn có thể troubleshoot graylog:
```sh
tailf /var/log/graylog-server/server.log
```
- Nếu bạn bắt được bản tin sau thì là ok
```sh
2016-06-01T21:26:05.689+07:00 INFO  [ServerBootstrap] Graylog server up and running.
```
<a name ="6"></a>
- Cài đặt
```sh
yum -y install graylog-web
```
- Tạo secret key giống với graylog-server
```sh
SECRET=$(pwgen -s 96 1)
sed -i -e 's/application\.secret=""/application\.secret="'$SECRET'"/' /etc/graylog/web/web.conf
```
- Cấu hình file configure và chỉnh sửa các tham số
```sh
vi /etc/graylog/web/web.conf
graylog2-server.uris="http://127.0.0.1:12900/"
timezone="Asia/Ho_Chi_Minh"
```
- Khởi chạy
```sh
systemctl restart graylog-web
```

<a name="7"></a>
### 7.Test
- Đăng nhập vào trình duyệt của bạn và nhập IP public của server
```sh
http://graylog_public_IP:9000/
```
- Nhập username `admin` và admin password `YourPassword`.
- Khi bạn login thành công sẽ hiện ra trang chủ :
<img src="http://i.imgur.com/5TjwvLq.png" />
- Bài viết đến đây là kết thúc, và chúng ta hãy vọc vạch nó thôi !!!

<a name="8"></a>
### 8.Tham khảo
- https://www.digitalocean.com/community/tutorials/how-to-install-graylog-1-x-on-centos-7
- http://www.itzgeek.com/how-tos/linux/centos-how-tos/how-to-install-graylog2-on-centos-7-rhel-7.htmls
