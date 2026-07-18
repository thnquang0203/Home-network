# Hướng dẫn cấu hình - Home Network trên Packet Tracer

Đây là tài liệu ghi lại cách mình đã dựng và cấu hình mạng gia đình này, theo đúng thứ tự đã làm thật trên Packet Tracer (không phải bản lý thuyết thuần túy, nên có vài chỗ khác so với dự tính ban đầu - mình ghi chú luôn lý do).

## Thiết bị dùng trong bài

- 1 Router **819HGW** (Home Gateway) - làm DHCP server, đóng vai router trung tâm
- 1 Switch **2960-24TT**
- 1 Access Point rời (**AccessPoint-PT**) - phát Wifi
- 2 PC (PC0, PC1)
- 1 Server (Server-PT) - chạy dịch vụ Web
- 1 Printer
- 2 Laptop (Laptop0, Laptop1) - cần gắn thêm card Wifi
- 1 Smartphone

**Lưu ý quan trọng:** Ban đầu mình định dùng luôn sóng Wifi tích hợp sẵn trong Router 819HGW (interface `wlan-ap0` / `Dot11Radio0`), cấu hình qua CLI bằng lệnh `service-module wlan-ap 0 session`. Cấu hình chạy được (log báo `Dot11Radio0 changed state to up`) nhưng Laptop không quét thấy sóng, và phiên console AP hay bị treo (`Connection refused by remote host`) không vào lại được để sửa tiếp. Sau vài lần thử không ăn, mình chuyển hẳn sang dùng **1 Access Point rời**, cắm dây từ Switch qua Port 0 của AP, cấu hình SSID/mật khẩu qua GUI - nhẹ nhàng hơn nhiều và chạy ngay lần đầu. Nếu bạn làm theo hướng dẫn này thì nên đi thẳng phương án AP rời luôn, đỡ mất thời gian.

## Sơ đồ đấu nối thực tế

```
                [819HGW Router0]
                       |
                  [Switch 2960]
              /      |      |      \
        Printer0  Server0  PC1    PC0
                       |
              [Access Point0] (nối dây từ Switch)
                  /         \
             Laptop0      Laptop1
             (Wifi)        (Wifi)

        Smartphone0 -- kết nối Wifi trực tiếp, không cần vẽ dây
```

## Địa chỉ IP thực tế sau khi cấu hình

DHCP pool khai báo dải `192.168.1.0/24`, loại trừ `192.168.1.1 - 192.168.1.10` để dành cho router và các IP tĩnh. Kết quả các máy nhận được:

| Thiết bị | IP | Cách gán |
|---|---|---|
| Router (Vlan1) | 192.168.1.1 | Tĩnh |
| Server0 | 192.168.1.10 | Tĩnh (đặt tay) |
| PC0 | 192.168.1.11 | DHCP |
| PC1 | 192.168.1.12 | DHCP |
| Printer0 | 192.168.1.20 | Tĩnh (đặt tay) |
| Laptop0 | 192.168.1.14 | DHCP qua Wifi |
| Smartphone0 | 192.168.1.15 | DHCP qua Wifi |
| Laptop1 | 192.168.1.17 | DHCP qua Wifi |

IP không theo thứ tự liền nhau (11, 12, 14, 15, 17...) là bình thường - DHCP cấp theo thứ tự thiết bị xin IP trước/sau, không theo thứ tự mình đặt tên máy.

**Wifi:**
- SSID: `HomeNet_WiFi`
- Bảo mật: WPA2-PSK, mã hóa AES
- Mật khẩu: `Home@12345`

## Các bước cấu hình

### 1. Router - đặt IP LAN

Cổng `FastEthernet0` trên 819HGW là cổng switch (Layer 2), không nhận lệnh `ip address` trực tiếp - phải cấu hình qua interface Vlan1:

```
enable
configure terminal
hostname HomeGateway
interface Vlan1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
```

Kiểm tra lại bằng `show ip interface brief`, dòng Vlan1 phải hiện `up up`.

### 2. Router - cấu hình DHCP

```
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp pool HOMENET
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
exit
end
copy running-config startup-config
```

### 3. Switch

Không cần cấu hình gì đặc biệt, dùng VLAN 1 mặc định. Đấu cáp Copper Straight-through từ Router (Vlan/FastEthernet nào có port vật lý) xuống Switch, rồi từ Switch tỏa ra các cổng còn lại tới PC0, PC1, Server0, Printer0, và Access Point0.

### 4. Access Point - cấu hình Wifi (làm qua GUI, không cần CLI)

Mở AccessPoint0 > tab Config > mục INTERFACE > Port 1:

- Port Status: On
- SSID: `HomeNet_WiFi`
- Authentication: WPA2-PSK
- PSK Pass Phrase: `Home@12345`
- Encryption Type: AES

Port 0 của AP là cổng dây (nối từ Switch), Port 1 là sóng Wifi - đừng nhầm hai cổng này.

### 5. PC0, PC1

Desktop > IP Configuration > chọn DHCP. Máy tự nhận IP, gateway, DNS.

### 6. Server0 - IP tĩnh

Desktop > IP Configuration > chọn Static:
- IP: `192.168.1.10`
- Subnet: `255.255.255.0`
- Gateway: `192.168.1.1`

Sau đó vào tab Services > HTTP > bật On để có Web server nội bộ (Server0 có sẵn vài trang html mặc định như `index.html`, dùng thử để test là đủ).

### 7. Printer0 - IP tĩnh

Vào Config > INTERFACE > FastEthernet0, chọn Static, điền:
- IP: `192.168.1.20`
- Subnet: `255.255.255.0`

Phần Gateway điền ở mục Global Settings riêng, không nằm chung với ô IP.

### 8. Laptop0, Laptop1 - gắn card Wifi

Laptop mặc định có card mạng dây. Để dùng Wifi:

1. Tab Physical, tắt nguồn máy (nút power nhỏ)
2. Kéo module card dây hiện có ra khỏi khe
3. Kéo module **WPC300N** vào khe đó (dùng đúng tên này - card `PT-LAPTOP-NM-1W` tuy cũng là Wifi nhưng bị công cụ "PC Wireless" báo lỗi không nhận, phải đổi sang WPC300N mới connect được)
4. Bật lại nguồn
5. Vào Desktop > PC Wireless > tab Connect > Refresh > chọn `HomeNet_WiFi` > nhập mật khẩu > Connect
6. Kiểm tra IP Configuration, interface đổi thành Wireless0 và tự nhận IP

### 9. Smartphone0

Không cần đổi card gì cả, mở IP Configuration là thấy interface Wireless0 tự bắt sóng và nhận IP luôn.

## Kiểm tra kết quả

Từ PC0 mở Command Prompt, ping thử sang máy dùng Wifi và Server:

```
ping 192.168.1.14
ping 192.168.1.10
```

Cả hai đều 0% loss - xác nhận hai nhánh dây và Wifi thông với nhau qua router/switch/AP bình thường.

Test thêm bằng Web Browser trên PC hoặc Laptop, gõ `http://192.168.1.10`, thấy trang mặc định của Server hiện ra là dịch vụ HTTP đã chạy đúng.

## Vài lỗi hay gặp lúc làm (để tham khảo nếu bị)

- **"Device must be powered on"** khi mở tab Config: quên bật nguồn máy ở tab Physical trước.
- **"Invalid input detected at '^' marker"** khi gõ `ip address` trên FastEthernet0 của 819HGW: do cổng đó là Layer 2, phải chuyển sang cấu hình trên `interface Vlan1`.
- **"AWMP300N or WPC300N wireless interface is required to connect"**: card `PT-LAPTOP-NM-1W` cắm đúng chỗ nhưng công cụ PC Wireless không nhận diện được, phải đổi sang module WPC300N.
- **"Connection refused by remote host"** khi vào `service-module wlan-ap 0 session`: phiên console vào AP nhúng trong router bị treo do trước đó thoát không đúng cách. Nếu gặp lỗi này lặp lại nhiều lần, đơn giản nhất là bỏ luôn AP nhúng, chuyển sang dùng Access Point rời như mình đã làm ở trên.
