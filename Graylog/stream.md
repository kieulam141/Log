	 	 	
# Streams

## Stream la gi

- Graylog stream là cơ chế để lọc gói tin trong thời gian thực khi nó đang dược xử lý.
- Chúng ta định nghĩa các rule để lọc gói tin.
## Use cases:
- Giám sát các lỗi.
- Lọc ra list SSh failed và sử dụng quick values để phân tích các user nào hay truy cập trái phép.

## Thao tác
- Chuyển hướng vào mục stream trên graylog.
- Create Stream: đặt title và descrpition.
- Click “Manage Rules”:
	+ Add stream rule:
		Field: các trường chính như: source, source-file, source-type, application-name, message, level, count …
		Type:
			match exactly: match chính xác.
			match regular expression: match theo biểu thức.
			greater than/ smaller than: lớn hoặc bé hơn.
			field presence ( không so sánh với value ): sự xuất hiện của field.
		Value: value để so sánh.
		Nếu muốn chọn các giá trị đối thì tích vào inverted.
		Description: …
+ Có 2 chế độ lọc message:
	AND: message thỏa mãn các rule AND với nhau
	OR: message thỏa mãn các rule OR với nhau
	+ Có 2 mục message để lọc:
		Lọc các message theo input đã tạo trong phần “Recent Message”
		Lọc theo ID message có được ở trang chủ.
	+ Sau khi chọn mục message thì click “Load Message” để xem các message có khớp với các rule đã tạo.
	+ I'm done để kết thúc.
- Click "Manage Alerts"
	+ Alerts dựa trên stream, mục đích chính tạo các trigger để gửi mail cảnh báo tới quản trị viên.
	+ VD: cảnh báo khi có 50 lần ssh login failed trong 1 minute.
	+ 3 điều kiện để tạo trigger:
		Đếm message: trigger khi stream nhận được nhiều hơn X message trong Y minute
		Field value: 
	

