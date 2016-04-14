# Các bài lab với DHCP
## Mục lục

[1.Cài đặt DHCP server] (#1)

[2.Cấu hình file configuration] (#2)

[a.Subnet Declaration] (#2a)

[b.Range Parameter] (#2b)

[c.Static IP Address Using DHCP] (#2c)

[d.Shared-network Declaration] (#2d)

[e.Group Declaration] (#2e)

[3.DHCP Relay Agent] (#3)

<a name="1"></a>
### 1.Cài đặt DHCP server
Trước tiên bạn cài đặt gói DHCP với quyền root.
<img src="http://i.imgur.com/8Ew5WCX.png" />

Sau khi cài, sẽ xuất hiện file configuration tại đường dẫn /etc/dhcp/dhcpd.conf, nhưng chỉ là file trống.
<img src="http://i.imgur.com/rXNdDXf.png" />

File configuration tương tự được lưu tại đường dẫn /usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample, bạn
có thể dùng file này để cấu hình file configuration của bạn vì nó hướng dẫn rất chi tiết.

<a name="2"></a>
### 2.Cấu hình file configuration
- Bước đầu tiên để cấu hình DHCP server là bạn phải tạo 1 file configuration ở đó lưu trữ thông tin về mạng
cho clients.Sử dụng file này để khai báo các tùy chọn riêng và chung cho hệ thống các clients.
- Có 2 kiểu báo cáo:
*Parameters(Các Thông số)* -- thể hiện cách thực hiện các tác vụ như thế nào, ở đâu và những tùy chọn
cấu hình network gửi đến clients là gì.
*Declarations(Các Khai báo)* -- mô tả cấu trúc của mạng, các clients, cung cấp địa chỉ cho các clients,
hoặc áp dụng 1 nhóm các thông số tới 1 nhóm các khai báo.
- Các thông số thường được bắt đầu bằng các từ khóa "options"
- Các thông số trước dấu "{}" được coi là thông số chung, áp dụng cho tất cả các phần bên dưới nó.
- Mô hình sau sẽ là mô hình dùng chung cho lab a,b,c.Các máy ảo đều sử dụng card VMnet1.
<img src="http://i.imgur.com/CIZAbfr.png" />

- Ở đây tôi dùng các máy ảo Centos2 làm dhcp server, Centos làm dhcp relay agent, 2 clients là win7 và win8(máy thật).

<a name="2a"></a>
#### a.Subnet Declaration
- Các tùy chọn về routers(default gateway), subnet-mask, domain-search, domain-name-servers, và time-offset
được sử dụng cho bất kì host nào được khai báo bên dưới.
- Với mỗi subnet được cấp và được kết nối tới DHCP server, nhất định phải có 1 subnet được khai báo để
cho DHCP server công nhận là có 1 địa chỉ trong subnet đó, kể cả subnet đó được khai báo rỗng không có địa chỉ 
thì nó sẽ được cấp động.
- Trong ví dụ này, có các tùy chọn chung cho các clients trong subnet được khai báo.Các clients được đăng kí
trong 1 địa chỉ ip nằm trong rải đã cho.
- Bạn khai báo vào file /etc/dhcp/dhcpd.conf như sau:
`subnet 10.0.0.0 netmask 255.255.255.0 {
	option routers					10.0.0.1;
	option subnet-mask 				255.255.255.0;
	option domain-search 			"example.com";
	option domain-name-servers 		10.0.0.1;
	option time-offset 				-18000;		#Eastern Standard Time
	range 10.0.0.11 10.0.0.20;
}`
<img src="http://i.imgur.com/HXwhXF4.png" />

- Sau đó start dịch vụ dhcp trên server.
- Xin cấp ip từ dhcp server của client.
<img src="http://i.imgur.com/bf2zS2I.png" />

- Kiểm tra trên client(win7)
<img src="http://i.imgur.com/1OaPS9S.png" />

<a name="2b"></a>
#### b.Range Parameter
- Trong ví dụ này DHCP server sẽ khai báo default lease time, maximum lease time và các giá trị cấu hình mạng
cho tất cả clients cùng hay # subnet(khác với ví dụ trên là áp dụng riêng cho các client trong cùng 1 subnet)
- Tương tự tôi khai báo vào file conf như sau:
`default-lease-time 600;
max-lease-time 7200;
option subnet-mask 255.255.255.0;
option broadcast-address 10.0.1.255;
option routers 10.0.1.1;
option domain-name-servers 10.0.0.1;
option domain-search "example.com";
subnet 10.0.1.0 netmask 255.255.255.0 {
   range 10.0.1.21 10.0.1.30;
}`
<img src="http://i.imgur.com/JYSJSrh.png" />

- Kiểm tra trên client
<img src="http://i.imgur.com/ytdn9Ne.png" />

<a name="2c"></a>
#### c.Static IP Address Using DHCP
- Để đăng kí địa chỉ ip dựa trên địa chỉ MAC của card mạng, tôi sử dụng thông số hardware ethernet cùng với
khai báo host.
- Với kiểu khai báo này thì card mạng có địa chỉ Mac 00-0C-29-FE-01-4E sẽ luôn nhận địa chỉ 10.0.1.32
`default-lease-time 600;
max-lease-time 7200;
option subnet-mask 255.255.255.0;
option broadcast-address 10.0.1.255;
option routers 10.0.1.1;
option domain-name-servers 10.0.1.1;
option domain-search "example.com";
subnet 10.0.1.0 netmask 255.255.255.0 {
   
}
host WIN7 {
   option host-name "win7.example.com";
   hardware ethernet 00:0C:29:FE:01:4E;
   fixed-address 10.0.1.32;
}
`
<img src="http://i.imgur.com/NOpapxF.png" />

<a name="2d"></a>
#### d.Shared-network Declaration
- Khi bạn muốn cấp nhiều ip cho nhiều subnet mạng của bạn, mà bạn chỉ có 1 DHCP server thì bạn sẽ dùng phương pháp này.
- Tiết kiệm được chi phí nhưng hiệu năng của dhcp server sẽ kém đi.
- Mô hình mạng như sau:
<img src="http://i.imgur.com/4OUyfVL.png" />

- Khai báo trên file conf của dhcp server:
<img src="">

<a name="2e"></a>
#### e.Group Declaration
- Khai báo group được sử dụng để áp các thông số chung cho nhóm đấy.
- Có thể là 1 nhóm shared-network, subnets hoặc các host.
- Ở đây tôi sẽ ví dụ về 1 nhóm các host.
`group {
   option routers                  10.0.1.1;
   option subnet-mask              255.255.255.0;
   option domain-search              "example.com";
   option domain-name-servers       10.0.1.1;
   option time-offset              -18000;     # Eastern Standard Time
   host win7 {
      option host-name "win7.example.com";
      hardware ethernet 00:A0:78:8E:9E:AA;
      fixed-address 10.0.1.56;
   }
   host win8 {
      option host-name "win8.example.com";
      hardware ethernet 00:50:56:C0:00:01;
      fixed-address 10.0.1.56;
   }
}
`
<img src="http://i.imgur.com/V2GY7Vi.png" />

- Kiểm tra trên client win7
<img src="http://i.imgur.com/ehPx2dt.png" />

- Kiểm tra trên client win8
<img src="http://i.imgur.com/CKaWqNp.png" />

<a name="3"></a>
### 3.DHCP Relay Agent
- Mô hình mạng như sau:
<img src="http://i.imgur.com/sK8QbiJ.png" />

- DHCP relay agent cho phép chuyển các yêu cầu dhcp và bootp từ 1 subnet ko có dhcp server trong đấy,
tới 1 hoặc nhiều dhcp server trên các subnet khác.
- Khi 1 client yêu cầu thông tin, DHCP relay agent chuyển yêu cầu đấy đến danh sách các dhcp server xác định
- Khi 1 DHCP server gửi lại reply, thì reply đó có thể là broadcast hoặc unicast gửi đến yêu cầu từ nguồn ban đầu.
- Mặc định DHCP server sẽ lắng nghe tất cả các yêu cầu DHCP từ tất cả các card mạng, trừ khi nó được chỉ định
trong /etc/sysconfig/dhcrelay với chỉ thị "INTERFACES".
- Giờ ta bắt đầu cấu hình DHCP relay agent.

#### a.Cấu hình trên dhcp server
- Set IP trên card VMnet2 nối với DHCP relay agent.
<img src="http://i.imgur.com/Cd954IA.png" />

- Cấu hình file conf để cấp ip cho subnet 1.
`subnet 10.0.0.0 netmask 255.255.255.0 {
	option routers					10.0.0.1;
	option subnet-mask 				255.255.255.0;
	option domain-search 			"example.com";
	option domain-name-servers 		10.0.0.1;
	option time-offset 				-18000;		#Eastern Standard Time
	range 10.0.0.11 10.0.0.20;
}`
<img src="http://i.imgur.com/DdxjUEC.png" />

- Cấu hình định tuyến tĩnh đến subnet 1 và kiểm tra lại bảng định tuyến.
`ip route add `
<img src="http://i.imgur.com/A3UOaBq.png" />

- Cuối cùng khởi động lại dịch vụ dhcp: service dhcpd restart.

#### b.Cấu hình trên dhcp relay agent
- Set IP trên card VMnet2 nối với DHCP server và VMnet1 nối với subnet 1.
<img src="http://i.imgur.com/SUNCu8N.png" />

- Chỉnh sửa file /etc/sysctl.conf.forward:ta chỉnh số, nếu có 1 server thì là 1, 2 server thì là 2.
<img src="http://i.imgur.com/SpT4FwW.png" />

- Chỉnh sửa file /etc/sysconfig/dhcrelay.
<img src="http://i.imgur.com/JBrxUTf.png" />

- Cuối cùng khởi động lại dịch vụ dhcrelay: service dhcrelay restart.

#### c.Kiểm tra ip trên client
- win7:
<img src="http://i.imgur.com/EPQWBKx.png" />

- win8:
<img src="http://i.imgur.com/dHINSTc.png" />


