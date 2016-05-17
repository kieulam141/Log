#System Auditing
## Mục lục
[1.Khái Niệm] (#1)

[2.Cài đặt và cấu hình dịch vụ] (#2)

[3.Phân tích file audit log]	(#3)

[4.Tham Khảo] (#4)

<a name="1"></a>
### 1.Khái Niệm
- Hệ thống audit trong linux cung cấp 1 cách để kiểm tra tất cả các thông tin trong hệ thống của bạn.
- Dựa vào các rules được cấu hình trước, Audit tạo ra các bản ghi log để ghi lại tất cả thông tin về các sự kiện xảy ra trong hệ thống của bạn nhiều nhất có thể.
- Nó không cung cấp các tính năng bảo mật nhưng nó có thể tìm ra các mối nguy hiểm cho hệ thống của bạn.
- Những thông tin mà Audit có thể ghi lại:
<ul>
	<li>Ngày giờ, loại và đầu ra của sự kiện.</li>
	<li>Danh tính của user đã kích hoạt sự kiện.</li>
	<li>Tất cả sự thay đổi cấu hình Audit và những nỗ lực truy cập file log Audit.</li>
	<li>Những sử dụng của cơ chế xác thực như SSH, Kerberos ...</li>
	<li>Những thay đổi đến bất kì database tin cậy như /etc/passwd.</li>
	<li>Những nỗ lực để nhập hoặc xuất thông tin vào hoặc ra từ hệ thống.</li>
	<li>Bao gồm hoặc loại trừ các sự kiện dựa trên định danh user, các labels vấn đề và đối tượng, và các thuộc tính khác.</li>
</ul>
- Các trường hợp sử dụng
<ul>
	<li>Xem file truy cập: Audit có thể kiểm tra 1 file hay folder đã được truy cập, chỉnh sửa, thực thi ở đâu.Nó dùng để đề phòng truy cập vào các file quan trọng.</li>
	<li>Giám sát system calls: Audit có thể ghi lại log mỗi khi 1 system call cụ thể được sử dụng.</li>
	<li>Ghi lại các câu lệnh được chạy bởi user.</li>
	<li>Ghi lại các sự kiện về bảo mật: Nó có thể ghi lại các nỗ lực login failed để cung cấp thông tin về user nào đang cố đăng nhập vào hệ thống.</li>
	<li>Tìm kiếm sự kiện: Audit cung cấp công cụ `ausearch`, dùng để lọc các bản tin log.</li>
	<li>Chạy các báo cáo tổng hợp: tool `aureport` có thể báo cáo chi tiết cho từng sự kiện, Admin có thể phân tích các báo cáo này.</li>
	<li>Giám sát Truy cập mạng: iptables và ebtables có thể kích hoạt sự kiện Audit, cho phép Admin giám sát truy cập mạng.</li>
</ul>

#### Cấu trúc hệ thống Audit
- Hệ thống Audit bao gồm 2 phần chính: các ứng dụng,tiện ích người dùng và tiến trình system call của kernel.
- Kernel nhận system calls từ các ứng dụng của người dùng và lọc chúng qua 1 trong 3 bộ lọc: user, task, hoặc exit.
- Khi 1 system calls qua 1 trong 3 bộ lọc trên, nó sẽ được gửi đến bộ lọc `Exclude`, dựa trên cấu hình rule Audit và được gửi tới Audit để ghi lại.
<img src="http://i.imgur.com/La2Lmm0.png" />

<a name="2"></a>
### 2.Cài đặt và cấu hình dịch vụ Audit
- Cài đặt dịch vụ: `yum -y install audit`
- Đường dẫn file config `/etc/audit/auditd.conf`.

#### Định nghĩa các rules trong file conf
- Các loại Audit rules:
<ul>
	<li>Control rules: cho phép thay đổi các hành động trong hệ thống Audit vào 1 số cấu hình.</li>
	<li>File system rules: cho phép giám sát truy cập tới file hoặc folder cụ thể.</li>
	<li>System call rules: cho phép log lại các system calls.</li>
</ul>
- Các rules được định nghĩa bởi command `auditctl` (chú ý rằng các rules này không được chạy mỗi khi hệ thống khởi động lại) hoặc viết vào file /etc/audit/audit.rules.

#### 2.1.Định nghĩa Rules với command `auditctl`
- Chú ý rằng tất cả các commands làm việc với Audit đều phải có quyền root.

##### Định nghĩa Control Rules
| Tham số | Ý nghĩa | Tùy chọn |
| ------- | ------- | ----- |
| -b | set tổng số buffer Audit lớn nhất trong kernel | --- |
| -f | set các hành động thể hiện khi 1 lỗi nguy hiểm được phát hiện | 0:im lặng ; 1:cảnh báo: 2:nguy kịch |
| -e | enables hoặc disables hệ thống Audit hoặc locks file configuration | 0:disables audit ; 1:enables audit ; 2:lock conf|
| -r | set tốc độ tạo ra bản tin mỗi giây | --- |
| -s | báo cáo trạng thái của hệ thống Audit | --- |
| -l | list ra toàn bộ rules | --- |
| -D | xóa toàn bộ rules | --- |
VD:
<img src="http://img.prntscr.com/img?url=http://i.imgur.com/0HKP21Z.png" />

##### Định nghĩa File System Rules
- Cú pháp:
```sh
auditctl -w path_to_file -p permissions -k key_name
```
<ul>
	<li>path_to_file: file hoặc folder sẽ được giám sát.</li>
	<li>permissions:các quyền được log:</li>
	<ul>
		<li>r: sự kiện đọc các truy cập tới file hoặc folder.</li>
		<li>w: sự kiện viết các truy cập tới file hoặc folder.</li>
		<li>x: sự kiện thực thi các truy cập tới file hoặc folder.</li>
		<li>a: thay đổi các đặc tính của file hoặc folder.</li>
	<ul>
	<li>key_name: là 1 xâu để định danh rule đấy.</li>
</ul>
- VD: định nghĩa 1 rule log lại tất cả sự kiện viết các truy cập và các thay đổi đặc tính của file /etc/passwd và tất cả file trong folder /etc/selinux
```sh
auditctl -w /etc/passwd -p wa -k passwd_changes
auditctl -w /etc/selinux/ -p wa -k selinux_changes
```
- Chú ý rằng xâu ở cuối câu lệnh phải là duy nhất.

##### Định nghĩa System Call Rules
- Cú pháp:
```sh
auditctl -a action,filter -S system_call -F field=value -k key_name
```
<ul>
	<li>action và filter: qui định khi nào 1 sự kiện nhất định được log.</li>
	<li>system_call: qui định system_call bởi tên của nó.</li>
	<li>field=value : qui định các tùy chọn thêm vào chỉnh sửa rule để match các sự kiện dựa trên 1 cấu trúc đặc biệt, group ID, process ID, ... .</li>
	<li>key_name : là 1 xâu để định danh rule đấy.</li> 
</ul>
- VD: định nghĩa rule để tạo các bản ghi mỗi khi system_calls adjtimex và settimeofday được sử dụng bởi chương trình bất kì, và hệ thống sử dụng cấu trúc 64-bit
```sh
auditctl -a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time_change
```

#### 2.2 Định nghĩa rule trong file /etc/audit/audit.rules
- Như đã nói ở trên khi định nghĩa rule bằng command `auditctl` thì khi hệ thống khởi động lại sẽ bị mất đi.
- Để giải quyết vấn đề trên thì chúng ta sẽ khai báo các rules trong file /etc/audit/audit.rules.
<img src="http://img.prntscr.com/img?url=http://i.imgur.com/FpJQPze.png" />
- Sau đó bạn restart lại dịch vụ : `service auditd restart`

<a name="3"></a>
#### 3.Phân tích file audit log
- Mặc định, hệ thống Audit sẽ lưu các log trong file /var/log/audit/audit.log.
- Chúng ta sẽ làm ví dụ nhỏ để phân tích các log được sinh ra trong file /var/log/audit/audit.log.
- Thêm rule để log mọi attempt (cố gắng) đọc hoặc chỉnh sửa file /etc/ssh/sshd_config
```sh
-w /etc/ssh/sshd_config -p warx -k sshd_config
```
- Sau đó chạy câu lệnh sau để tạo sự kiện mới trong file log Audit.
```sh
cat /etc/ssh/sshd_config
```
- Tiếp theo đọc file audit.log để phân tích các sự kiến mới:
```sh
tail -f /var/log/audit/audit.log
```
<img src="http://img.prntscr.com/img?url=http://i.imgur.com/xI0GGP7.png" />

- Sự kiện trên gồm 3 bản ghi (bắt đầu bằng type=keyword)
##### First Record
- type=SYSCALL: trường type bao gồm loại bản ghi.Xem các loại bản ghi tại đường dẫn `https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sec-Audit_Record_Types.html`
- msg=audit(1463413606.335:43): ghi lại time stamp(1463413606), ID duy nhất của bản ghi(335).Nhiều bản ghi có thể có cùng time stamp và ID nếu chúng được tạo ra như 1 phần cùng 1 sự kiện.
- arch=c000003e: bao gồm thông tin về cấu trúc CPU.
- syscall=2: ghi lại loại system call được gửi tới kernel.
- success=yes: xác nhận xem system call được ghi lại trong 1 sự kiện cụ thể có thành công hay thất bại.
- items=1: bao gồm số đường dẫn bản ghi trong sự kiện.
- ppid=1679: ID của tiến trình cha.
- pid=1761: ID của tiến trình chính.
- auid=0: ID của user Audit.
- uid=0, gid=0: ID của user và nhóm của user đã chạy tiến trình.
- tty=pts0: ghi lại terminal đã gọi tiến trình.
- ses=1: ID của session.
- comm="cat": ghi lại tên của command đã gọi tiến trình.
- key="sshd_config": keyword của rule.

##### Second Record
- type=CWD (current working directory) ghi lại folder làm việc nơi mà tiến trình gọi system call trong bản ghi thứ 1.
- msg=audit(1463413606.335:43) : giống với bản ghi thứ 1.
- cwd="/root" : đường dẫn đến thư mục mà system call được gọi.

##### Third Record
- type=PATH: ghi lại mỗi đường dẫn mà nó đi qua.
- msg=audit(1463413606.335:43): giống với bản ghi thứ 1.
- item=0: chỉ ra item trong tổng số các items trong bản ghi loại SYSCALL, giá trị=0 nghĩa là nó là item thứ 1.
- name="/etc/ssh/sshd_config": ghi lại đường dẫn đầy đủ của file hoặc folder.
- inode=785850: chứa số inode liên quan đến file hoặc folder.
- dev=fd:00: qui định ID chính và phụ của device bao gồm file hoặc folder được ghi lại trong sự kiện.
- mode=0100600: ghi lại quyền của file hoặc folder, được mã hóa bằng kí hiệu số. trong trường hợp này là 0100600 : -rw-------.

<a name="4"></a>
### 4.Tham Khảo
- https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sec-Understanding_Audit_Log_Files.html
