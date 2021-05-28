Ví dụ cho stm32fd103c8t6 (blue pill)
chương trình này redirect hàm printf của stdio để gửi chuỗi ra serial UART. Bật tắt LED tại PC13 (GPIOC 13) mỗi 1 giây, led bật sẽ gửi chuỗi "led on" và ngược lại gửi chuỗi "led off" ra serial UART
