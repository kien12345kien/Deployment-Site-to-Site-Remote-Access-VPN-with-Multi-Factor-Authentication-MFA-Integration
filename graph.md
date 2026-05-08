### 1. Biểu đồ "Network Traffic Basic" (Băng thông mạng)

- **Vị trí trên Grafana:** Nằm ngay phần Network.

- **Giá trị chứng minh:** Biểu đồ này sẽ vẽ 2 đường (Receive và Transmit). Khi em chạy bài test 9 Clients, đường biểu đồ này sẽ vọt lên.

- **Cách phân tích trong báo cáo:** Em chụp lại lúc nó đạt đỉnh (Peak) khoảng 250 - 280 Mbps và đi ngang (flatten out). Biểu đồ này thay thế hoàn hảo cho những con số log khô khan của `iperf3`, trực quan hóa việc Gateway của em đã "chạm trần" giới hạn xử lý mạng như thế nào.


### 2. Biểu đồ "CPU Basic" (Tải CPU tổng thể & Cấu trúc tải)

- **Vị trí trên Grafana:** Nằm ở hàng đầu tiên hoặc phần CPU.

- **Giá trị chứng minh:** Thay vì chỉ vẽ 1 đường CPU Load chung chung, biểu đồ này bóc tách CPU thành nhiều màu: Xanh lá (`%user`), Vàng (`%system`), Đỏ (`%iowait`), v.v.

- **Cách phân tích trong báo cáo:** Đây là vũ khí để em chứng minh luận điểm **Context Switching (Chuyển đổi ngữ cảnh)**. Em hãy chỉ cho Hội đồng thấy vùng màu Vàng (`%system`) phình to bất thường so với vùng màu Xanh (`%user`). Nó chứng tỏ nhân Kernel (system) đang phải làm việc cật lực để luân chuyển gói tin mạng qua lại, chứ không phải CPU cạn kiệt do thuật toán mã hóa (user).


### 3. Biểu đồ "CPU Interrupts" hoặc "CPU Usage per Core" (Tải từng nhân CPU)

- **Vị trí trên Grafana:** Trong mục CPU, thường có các biểu đồ chia nhỏ CPU 0, CPU 1...

- **Giá trị chứng minh:** Biểu đồ này vẽ riêng biệt mức tải của từng nhân.

- **Cách phân tích trong báo cáo:** Đây chính là "Bằng chứng thép" để kết tội môi trường máy ảo (VM) vô hiệu hóa `irqbalance`. Em hãy chụp khoảnh khắc đường biểu đồ của **CPU 0** dựng đứng chạm nóc (vì phải gánh toàn bộ ngắt mạng - Interrupts), trong khi đường của **CPU 1** lại nằm im lìm ở dưới đáy. Nó giải thích hoàn hảo cho nút thắt "Single-Queue" của card mạng ảo.

### 4. Biểu đồ "Context Switches" (Số lần chuyển ngữ cảnh)

- **Vị trí trên Grafana:** Thường nằm ở mục System hoặc Hardware.

- **Giá trị chứng minh:** Nó đếm số lần CPU phải chuyển trạng thái xử lý trong 1 giây.

- **Cách phân tích trong báo cáo:** Khi OpenVPN chạy đa luồng, con số này sẽ tăng dựng đứng. Em dùng biểu đồ này để kết luận: _"Bản chất OpenVPN chạy ở User-space đã tạo ra quá nhiều Context Switches khi luân chuyển dữ liệu, khiến CPU bị quá tải ở khâu điều phối chứ chưa kịp mã hóa xong, dẫn đến rớt băng thông"_.