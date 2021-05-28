# pi2stm32
Sử dụng Raspberry Pi để nạp (flash) code cho STM32 thông qua cổng serial UART. Không cần mạch nạp.
Dựa trên bài blog của Matthew Dunn [tại đây](https://siliconjunction.wordpress.com/2017/03/21/flashing-the-stm32f-board-using-a-raspberry-pi-3/) 
Các bước được thực hiện trên Raspberry Pi 3 model B
# Chuẩn bị
- Cài đặt stm32flash trên Ras Pi:
```Shell
sudo apt-get install stm32flash
```
(thông tin chi tiết của stm32flash xem [tại đây](https://sourceforge.net/p/stm32flash/wiki/Home/))
- Sử dụng full UART ở GPIO thay vì Bluetooth:
  - Sửa file /boot/config.txt:
  ```Shell
  sudo nano /boot/config.txt
  ```
  - Thêm dòng sau vào cuối:
  ```Shell
  dtoverlay=pi3-miniuart-bt
  ```
- Chặn console chạy ở Serial port:
  - Sửa file /boot/cmdline.txt
  ```Shell
  sudo nano /boot/cmdline.txt
  ```
  - Xoá chuỗi: `console=serial0,115200`  
- Khởi động lại Ras Pi
- Kết nối STM32 với Ras Pi như sau:

<table>
  <tr><td>STM32</td><td>Raspberry Pi</tr>
  <tr><td> 3.3V </td><td> 3.3V (pin 1) </tr>
  <tr><td> G </td><td> GND (pin 2) </tr>
  <tr><td> A9 (TX) </td><td> GPIO15 (RX) (pin 10) </tr>
  <tr><td> A10 (RX) </td><td> GPIO14 (TX) (pin 8) </tr>
  <tr><td> R </td><td> GPIO2 (pin 3) </tr>
  <tr><td> BOOT0 </td><td> GPIO3 (pin 5) </tr>
</table>
<img src=https://user-images.githubusercontent.com/29064137/119982805-7ff54e00-bfe9-11eb-9a9a-620a29ac8fcf.png width = 500>

# Tiến hành nạp code
## Build chương trình từ Keil
Để nạp code thì tất nhiên là phải có code. Ta sử dụng file .hex đầu ra sau khi build từ Keil để nạp.
Đảm bảo rằng Keil output ra file .hex sau khi build:

![image](https://user-images.githubusercontent.com/29064137/119989620-969fa300-bff1-11eb-9c7a-24e81a961303.png)

Ấn F7 để build.

Sau khi build xong, gửi file .hex tới Ras Pi, ví dụ gửi thông qua VNC bằng VNC Viewer:

![image](https://user-images.githubusercontent.com/29064137/119990688-d5822880-bff2-11eb-83cc-8bf744e2821b.png)

## Nạp bằng stm32flash trên Ras Pi
- Mở terminal gõ `stm32flash -h` để hiển thị hướng dẫn sử dụng
- Để nạp, ta dùng lệnh sau:
```Shell
stm32flash -b 115200 -R -i 3,-2,2:-3,-2,2 -v -w /path/to/hex /dev/serial0
```
  Trong đó:
  - `-b 115200` để đặt baud rate là 115200
  - `-R -i 3,-2,2:-3,-2,2` để đặt chu trình khởi động và reset. Cụ thể 3,-2,2 tức là GPIO3=BOOT0=HIGH -> GPIO2=R=LOW -> GPIO2=R=HIGH, đây là trình tự để reset stm32 về bootloader trước khi nạp. -3,-2,2 tương đương GPIO3=BOOT0=LOW -> GPIO2=R=LOW -> GPIO2=R=HIGH, đây là trình tự để reset stm32 về bộ nhớ flash để chạy chương trình sau khi nạp.
  - `-v` để xác thực ghi (nạp)
  - `-w /path/to/hex` để ghi (nạp) file hex ở đường dẫn /hex/file/path 
  - `/dev/serial0` là thiết bị ở serial0 của Ras Pi, ở đây là stm32 được nối tới GPIO14 và GPIO15
## Tuỳ chọn: đơn giản hoá cú pháp cho lệnh nạp
- Lệnh nạp của stm32flash trên khá dài dòng, trừ đường dẫn file hex ra thì những tham số còn lại đều là cố định. Để đơn giản hoá cú pháp, mình đã viết một shell script thay đổi cú pháp. Với script này thì chỉ cần gõ `pi2stm32 /path/to/hex` trên terminal là có thể nạp cho stm32 được rồi.
- Lấy script về:
```Shell
wget 
chmod +x pi2stm32
sudo cp pi2stm32 /usr/local/bin
```
