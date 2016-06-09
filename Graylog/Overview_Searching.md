# Searching
## I.Search query language
### 1.Syntax
<img src="http://image.prntscr.com/image/7c1b81e00fb44eceafb9f369331f2e15.png" />

- Các cú pháp tìm kiếm rất giống cú pháp Lucene.
- Mặc định toàn bộ trường message được tìm kiếm nếu bạn không chỉ định trường message được tìm kiếm.
- Messages bao gồm chỉ định ssh: `ssh`
- Messages bao gồm chỉ định ssh hoặc login: `ssh login`
- Messages bao gồm chính xác cụm từ  *ssh login*: `"ssh login"`
- Messages mà trường "type" bao gồm ssh: `type:ssh`
- Messages mà trường "type" bao gồm chính xác cụm từ *ssh login*: `type:"ssh login"`
- Messages không có trường "type": `_missing_:type`
- Messages có trường "type": `_exists_:type`
- Mặc định thì các chỉ định hoặc cụm từ đều là kết nối OR.Bạn có thể dùng toán tử # để điều khiển theo ý mình.
```sh
"ssh login" AND source:example.org
("ssh login" AND (source:example.org OR source:another.example.org)) OR _exists_:always_find_me
```
- Bạn cũng có thể dùng toán tử NOT:
```sh
"ssh login" AND NOT source:example.org
NOT example.org
```
* Chú ý rằng, các toán tử bắt buộc phải viết hoa *
- Wildcards: sử dụng ? để thay thế 1 kí tự hoặc * để thay thế 0 hoặc nhiều kí tự.
```sh
source:*.org
source:exam?le.org
source:exam?le.*
```
- Chú ý rằng wildcards bị disabled để tránh quả tải bộ nhớ. Bạn có thể enable chúng trong graylog-server.conf: `allow_leading_wildcard_searches = true`
- Trường numeric hỗ trợ truy vấn theo 1 dải.
```sh
http_response_code:[500 TO 504]
http_response_code:{400 TO 404}
bytes:{0 TO 64]
http_response_code:[0 TO 64}
```
- Bạn cũng có thể tìm kiếm 1 phía:
```sh
http_response_code:>400
http_response_code:<400
http_response_code:>=400
http_response_code:<=400
```

- Và có thể kết hợp với các toán tử:
```sh
http_response_code:(>=400 AND <500)
```

## 2.Time frame selector
- Bộ chọn khung thời giản chỉ ra khoảng thời gian cần tìm kiếm.
- Nó cung cấp 3 cách chọn và sẽ có tốc độ tìm kiếm # nhau.

### 2.1.Relative time frame selector
<img src="http://image.prntscr.com/image/ba7e6f36001b4005a8ccba7c1ae3b4b9.png" />
- Bộ chọn này cho phép bạn tìm kiếm messages từ các tùy chọn đã được chọn tùy thuộc vào thời gian bạn chọn trong search button.
- VD: trong 5 phút, 30 phút, 1 tiếng , 2 tiếng ... gần nhất.

### 2.2.Absolute time frame selector
<img src="http://image.prntscr.com/image/dfab2ef579964783b32ae64788c77c8e.png" />
- Khi bạn biết chính xác khoảng thời gian bạn muốn tìm, bạn sẽ dùng bộ chọn này.
- Đơn giản là bạn chỉ cần nhập ngày và thời gian trong lịch có sẵn trên giao diện.

### 2.3.Keyword time frame selector
<img src="http://image.prntscr.com/image/270ea3f1344d43b689325831c23b8aa2.png" />

- Bộ chọn này cung cấp cho bạn keyword cho phép bạn tùy chỉnh tìm kiếm theo ý của bạn, và ngôn ngữ được sử dụng cũng là ngôn ngữ tự nhiên.

### 3.Saved searches
- Nếu bạn muốn lưu lại tìm kiếm để sử dụng sau này, thì Graylog cung cấp chức năng lưu tìm kiếm.
- Khi bạn submit xong tìm kiếm của bạn, lựa chọn các trường mà bạn muốn show từ sidebar search, sau đó save lại.
<img src="http://image.prntscr.com/image/9abc555119554684995dfcccb16bbb2c.png" />

- Sau đó đặt tên.
<img src="http://image.prntscr.com/image/5eae18d83f18405baaba1a6b8678341b.png" />

- Sau khi đã lưu, nếu bạn muốn xem lại thì vào mục sau:
<img src="http://image.prntscr.com/image/5eae18d83f18405baaba1a6b8678341b.png" />

- Bạn có thể update tên đã lưu, hoặc xóa kết quả tìm kiếm mình đã lưu
<img src="http://image.prntscr.com/image/b8a1ce9c11b64b0687760ab47cdff134.png" />

## II.Analysis
<img src="http://image.prntscr.com/image/b99b427f862f43828165d7884d95fade.png" />
- Graylog cung cấp 1 vài tool để phân tích kết quả tìm kiếm của bạn.
- Có 3 tool chính như sau:

### 1.Field statistics
- Tính toán các số liệu # nhau giữa các trường của bạn giúp bạn có cái nhìn tổng quan và hiểu các dữ liệu có trong chúng.
- Các thông tin bao gồm: total, Mean, Minimum, maximum, standard deviation (độ lệch tiêu chuẩn), variance (sự # nhau), sum, và cardinality.
<img src="http://image.prntscr.com/image/cc2fe8b2ae094b8ab21cd9f087d9d428.png" />

- Ngoài ra bạn cũng có thể ẩn, ngừng reload và thêm vào dashboard.

### 2.Quick values
- Tool này giúp bạn nhìn ra được sự phân phối các giá trị của các trường.
- Bên cạnh biểu đồ hình tròn là 1 bảng với những giá trị # nhau, cho phép bạn xem những lần chúng xuất hiện
<img src="http://image.prntscr.com/image/c39636adb89142abb8386064c2bd01aa.png" />

### 3.Field graphs
- Bạn có thể các đồ thị cho các trường loại số
<img src="http://image.prntscr.com/image/58a6548ff7774a7a9b30c188d0e57168.png" />

- Trong phần customize có các trường # nhau nữa:
<ul>
	<li>Value (các giá trị hiển thị trên đồ thị): mean, min, max, sum, total, cardinality</li>
	<li>Type (cách biểu diễn đồ thị): Area, Bar, Line.</li>
	<li>Interpolation (Tìm hiểu sau)</li>
	<li>Resolution (Khoảng time): Minute, Hour, Day, Week ...</li>
</ul>
- Và 1 tính năng rất hay nữa là bạn có thể so sánh trực quan các giá trị # nhau của cùng 1 trường trên cùng 1 đồ thị.
- Nhấn vào biểu tượng hamburger vào kéo nhập xuống đồ thị cần so sánh, ở đây t sẽ so sánh 2 giá trị là min và max
<img src="http://image.prntscr.com/image/81cebe79798043488a44b5d80ca2c6f2.png" />

- Và kết quả:
<img src="http://image.prntscr.com/image/736c04c8f16b48bcb91840a3a4692262.png" />

## IV.Export results as CSV
- Thêm 1 tính năng nữa rất hay là Graylog cho phép bạn export kết quả tìm kiếm ra 1 file CSV.
- Bạn chọn tất cả các trường cần export trong sidebar, sau đó vào phần *More actions* và chọn *Export as CSV*.
<img src="http://image.prntscr.com/image/514b9243b94d42efa070b29f5c304db1.png" />

- Bạn có thể mở file đã export bằng excel
<img src="http://image.prntscr.com/image/28dadaa42179461791a717035af3c4ed.png" />

## V.Search result highlighting

