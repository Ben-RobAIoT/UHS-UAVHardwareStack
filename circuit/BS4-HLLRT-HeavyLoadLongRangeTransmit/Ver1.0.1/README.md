## 1. Khối nguồn (XT30 3S-4S → TPS54302 → 5V)

**Chia áp hồi tiếp R1=100k / R2=13.3k: TÍNH ĐÚNG.** Nhiều người nhầm Vref của TPS54302 là 0.8V (dòng TPS543x khác), nhưng thực tế theo datasheet TI, TPS54302 dùng ngưỡng so sánh nội bộ là **0.596V**. Với công thức Vout = Vref×(1 + R1/R2):
- Vout = 0.596 × (1 + 100k/13.3k) = 0.596 × 8.519 ≈ **5.08V** — nằm gọn trong dải 4.75–5.25V mà module HP-H09-01 yêu cầu. Chuẩn.

**Vấn đề lớn nhất – ngân sách dòng (current budget):** Đây là chỗ bạn cần xem lại kỹ nhất. TPS54302 giới hạn **3A liên tục**. Theo bảng dòng tiêu thụ trong datasheet module, ở chế độ VHT80 MCS0 TX 2TX RF test, module có thể peak tới **1.79A**, và Radxa khuyến nghị tối thiểu **5V/2A** cho ZERO 3W (theo docs chính thức của Radxa). Nếu +5V ra khỏi TPS54302 vừa nuôi module WiFi vừa cấp ngược lên rail 5V của header (đúng như cách RPi/Radxa-compatible header hoạt động – pin 5V trên header nối thẳng vào rail nguồn chính của board), thì **tổng dòng peak lý thuyết có thể chạm 3.5–4A**, vượt giới hạn an toàn của TPS54302 → dễ vào chế độ hiccup/OCP giữa chuyến bay, đúng lúc cần WiFi ổn định nhất.

Khuyến nghị: hoặc (a) đổi sang buck 4–5A (ví dụ TPS54331/TPS563201 công suất lớn hơn, cùng họ dễ thay chân), hoặc (b) xác nhận rõ ràng: HAT này chỉ nuôi module WiFi, còn Radxa vẫn được cấp nguồn độc lập qua cổng Type-C riêng của nó (không đấu chung rail 5V qua header). Cách (b) an toàn hơn nhưng cần bạn chốt lại ý đồ thiết kế.

**Thiếu bảo vệ đầu vào pin:** XT30PW-M nối thẳng pin 3S-4S vào VIN của TPS54302, chưa thấy:
- Bảo vệ ngược cực (diode/MOSFET reverse protection) – rất nên có với pack pin tháo lắp trên UAV, cắm nhầm cực là cháy mạch.
- Cầu chì/PTC trên đường VIN.
- Tụ bulk điện phân gần connector để hấp thụ xung dòng do ESC/motor gây nhiễu trên rail pin chung (UAV thường share pack pin giữa ESC và companion computer).

**L1 4.7uH:** giá trị hợp lý ở 400kHz, nhưng bạn cần chọn inductor có **dòng bão hòa (Isat) > 4A** để có margin, vì dòng đỉnh cuộn cảm ≈ Iout + ΔIripple/2, và nếu bạn tăng công suất buck theo mục trên thì Isat cần theo tương ứng.

**EN pin (chân 5 của U2):** trong netlist mình không thấy điện trở pull-up nào nối EN lên VIN. TPS54302 cần EN được kéo lên mức cao để tự khởi động (không có pull-up nội bộ đảm bảo an toàn), nếu để nổi có thể mạch không khởi động ổn định. Kiểm tra lại xem có pull-up ẩn ở đâu chưa hoặc thêm 1 điện trở ~100k EN→VIN.

## 2. Khối USB2.0 tín hiệu

**ESD protection D1 = USBLC6-2SC6:** đây là lựa chọn phổ biến và hoạt động tốt cho USB2.0 HS trên thực tế, NHƯNG cần lưu ý: datasheet HP-H09-01 ghi rõ khuyến nghị **"equivalent capacitance value of ESD protection TVS is less than 1pF"**, trong khi USBLC6-2SC6 có **line capacitance tối đa 3.5pF** (theo datasheet ST). Đây là chênh lệch đáng kể so với khuyến nghị của chính hãng module. Trong thực tế nhiều thiết kế vẫn dùng USBLC6-2 cho USB2.0 480Mbps mà chạy ổn (vì USB2.0 HS khoan dung hơn USB3.0 SuperSpeed với capacitance), nhưng nếu bạn muốn bám sát khuyến nghị của B-Link 100%, nên tìm ESD array có Ctyp dưới 1pF (một số dòng như PESD5V0X1BL, hoặc ESD chuyên cho USB3.0/HS có cap thấp hơn). Đây là điểm cân nhắc rủi ro/lợi ích chứ không phải lỗi chết người.

**Kết nối Type-C 16-pin (USB3 connector):** vì đây là Type-C chỉ chạy USB2.0 (không có SuperSpeed TX/RX), bạn *bắt buộc* phải đấu chập cặp Dp1(A6)–Dp2(B6) và Dn1(A7)–Dn2(B7) lại với nhau trên PCB, vì connector Type-C có 2 cặp D+/D- đối xứng cơ khí để hỗ trợ cắm lật mặt, nhưng chỉ 1 cặp thực sự nối tới IC tuỳ hướng cắm. Nếu chưa chập, cắm lật đầu sẽ mất tín hiệu. Kiểm tra lại phần này trong file gốc (netlist ở đây không cho thấy rõ 2 cặp đó có chung net hay không).

**VBUS trên Type-C (A4/B9, B4/A9):** cần xác nhận VBUS ở đây **không** được nối vào rail +5V bạn tạo ra từ TPS54302 – nếu nối chung, khi cắm sợi cáp USB-C khác vào (hoặc khi Radxa cấp VBUS ngược qua cổng OTG) sẽ có 2 nguồn 5V "đấu đá" nhau trên cùng 1 rail, có thể gây back-feed hỏng nguồn. Cách an toàn: để VBUS pin đó không kết nối (NC) nếu chỉ dùng connector này để truyền D+/D-, hoặc thêm diode ORing nếu cố tình muốn có thể cấp nguồn qua đó.

**Trở CC1/CC2 = 5.1kΩ pull-down:** đúng chuẩn để khai báo thiết bị là UFP/sink mặc định 5V – hợp lý cho việc mô-đun WiFi hoạt động như thiết bị USB được Radxa nhận diện.

## 3. Header 2×20 (H1) – kết nối cơ khí kiểu HAT

Bạn chỉ route +5V, GND, WIFI_RST qua header — hợp lý về nguyên tắc (vì D+/D- không nằm trên header chuẩn Pi/Radxa). Nhưng:
- Cần đảm bảo **nhiều chân GND** trên header được nối vào plane GND chung (không chỉ 1–2 chân), để giảm loop nhiễu và tạo đường hồi lưu dòng tốt — đặc biệt quan trọng khi mạch này sẽ rung động liên tục trên khung UAV.
- **WIFI_RST**: theo datasheet, RESET đã có pull-up nội bộ 100k lên 3.3V sau điện trở nối tiếp 10K bên trong module — nghĩa là bạn **không cần** thêm pull-up ngoài. Nhưng bạn cần làm rõ: đây là nút nhấn cơ khí (cần thêm tụ debounce nhỏ, ví dụ 100nF từ RESET xuống GND để chống nhiễu/nảy phím) hay là tín hiệu điều khiển từ GPIO của Radxa (phần mềm reset)? Hai cách đi dây khác nhau, và trong UAV rung động mạnh thì nút nhấn cơ khí dễ gây reset giả nếu không debounce tốt.

## 4. Layout & EMI – quan trọng nhất vì bạn tự nhận đây là mạch "dễ nhiễu"

- Giữ đúng khuyến nghị datasheet: cặp vi sai USB2.0-DP/DM phải đi **90±5Ω differential**, dài **càng ngắn càng tốt**, có GND bao quanh (guard trace hoặc via-stitched ground pour hai bên).
- Đặt USBLC6-2SC6 **càng gần connector Type-C càng tốt** (nguyên lý ESD: chặn xung càng sớm càng ít ký sinh cảm kháng lan vào IC).
- Tách xa: cặp USB HS không nên đi gần node chuyển mạch (SW pin) của TPS54302 – switching tại 400kHz phát nhiễu bức xạ khá mạnh, nên giữ khoảng cách hoặc có plane GND chắn giữa.
- Antenna J0/J1 (IPEX) nằm ngay trên module – khi stack thành HAT, chú ý độ hở cơ khí cho dây ăng-ten không bị đè bởi lớp Radxa phía dưới/trên, và định tuyến dây ăng-ten tránh chạy song song sát rail công suất/switching để giảm suy hao & nhiễu xen (đúng như cảnh báo "avoid interference from Power and other signals" trong mục 6.2.2 datasheet).
- Vùng GND pad dưới module (theo mục 6.6 – tản nhiệt) cần via xuống layer dưới đủ dày, vì FEM công suất cao của module này **tự sinh nhiệt đáng kể**, và tài liệu khuyến cáo rõ phải bổ sung heat-sink/thermal pad khi hoạt động ở công suất tối đa – việc này dễ bị bỏ qua khi thiết kế dạng HAT xếp chồng vì không gian tản nhiệt bị giới hạn giữa 2 board.

## Tóm tắt ưu tiên sửa

1. **Xác nhận/tính lại ngân sách dòng tổng của TPS54302** nếu rail 5V cấp cả Radxa lẫn module — đây là rủi ro hệ thống lớn nhất.
2. Thêm bảo vệ ngược cực + fuse/PTC ở đầu vào pin XT30.
3. Kiểm tra pull-up cho chân EN của TPS54302.
4. Xác nhận VBUS của Type-C không đấu chồng vào rail +5V bạn tạo ra.
5. Chập Dp1-Dp2 / Dn1-Dn2 nếu Type-C chỉ chạy USB2.0.
6. Cân nhắc ESD có capacitance thấp hơn nếu muốn bám sát khuyến nghị <1pF của B-Link (không bắt buộc, USBLC6-2 vẫn chạy được).
7. Làm rõ cơ chế RESET (nút nhấn vs GPIO) để chọn linh kiện debounce phù hợp.
8. Tăng cường GND stitching quanh cặp USB HS và tính toán lại tản nhiệt cho FEM khi stack.
