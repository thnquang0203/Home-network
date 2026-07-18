# Home Network — Mô phỏng mạng gia đình bằng Cisco Packet Tracer

Dự án mô phỏng một mạng gia đình hoàn chỉnh (có dây + không dây) sử dụng Cisco Packet Tracer, gồm: Home Gateway Router (DHCP/NAT/Wifi), Switch, Server nội bộ, Printer, và các thiết bị Wifi (Laptop, Smartphone).

## Sơ đồ tổng quan

```
                [819HGW Router0]
                       |
                  [Switch 2960]
              /      |      |      \
        Printer0  Server0  PC1    PC0
                       |
              [Access Point0] -- SSID: HomeNet_WiFi
                  /         \
             Laptop0      Laptop1
             (Wifi)        (Wifi)

        Smartphone0 -- kết nối Wifi trực tiếp
```

Router có Wifi tích hợp (`wlan-ap0`) nhưng khi cấu hình thật bị treo phiên console, không vào lại được để chỉnh sửa. Nên bản này dùng 1 Access Point rời cắm dây từ Switch, cấu hình qua GUI - chạy ổn định ngay lần đầu. Lý do cụ thể và các lỗi gặp phải nằm trong file hướng dẫn.

Chi tiết đầy đủ về địa chỉ IP, cách đấu nối và cấu hình từng thiết bị nằm trong [`docs/huong-dan-cau-hinh.md`](docs/huong-dan-cau-hinh.md).

## Cấu trúc repo

```
home-network-packet-tracer/
├── README.md                     # File này
├── LICENSE
├── .gitignore
├── packet-tracer/
│   └── home-network.pkt          # File dự án Packet Tracer (mở bằng PT 8.x trở lên)
├── docs/
│   └── huong-dan-cau-hinh.md     # Hướng dẫn chi tiết + bảng IP thực tế + lỗi hay gặp
├── configs/
│   └── switch-config.txt         # Lệnh CLI cấu hình Switch (backup running-config)
└── screenshots/
    └── (ảnh chụp topology, kết quả ping, DHCP, v.v.)
```

## Yêu cầu

- Cisco Packet Tracer 8.2 trở lên.
- Không cần license đặc biệt, chỉ cần tài khoản Cisco NetAcad (miễn phí) để tải phần mềm.

## Cách sử dụng

1. Clone repo:
   ```
   git clone https://github.com/<username>/home-network-packet-tracer.git
   ```
2. Mở file `packet-tracer/home-network.pkt` bằng Cisco Packet Tracer.
3. Đọc `docs/huong-dan-cau-hinh.md` để hiểu cách dựng lại từ đầu hoặc đối chiếu cấu hình.

## Kết quả kiểm tra

- [x] Các thiết bị có dây nhận đúng IP tĩnh/DHCP
- [x] Laptop/Smartphone kết nối được Wifi và nhận IP qua DHCP
- [x] Ping thành công giữa các thiết bị trong LAN (test PC0 → Laptop0 Wifi và PC0 → Server0, cả hai 0% loss)
- [x] Truy cập được Web server nội bộ (192.168.1.10) từ cả máy dây lẫn máy Wifi
- [ ] Ping ra Internet giả lập (chưa dựng Cloud-PT trong bản này, có thể bổ sung sau)

Ảnh chụp minh chứng đặt trong `screenshots/`.

## Tác giả

- Họ tên:
- Lớp/Môn học:
- Ngày thực hiện:

## License

Xem [LICENSE](LICENSE).
