# UAV 5G WiFi Integration HAT for Radxa Zero 3W
VERVSION: 1.0.2

## 📖 Tổng quan dự án (Project Overview)
Đây là tài liệu thiết kế phần cứng cho bo mạch mở rộng (HAT) tích hợp module WiFi 5G, được thiết kế dưới dạng xếp chồng (stack) lên máy tính nhúng Radxa Zero 3W. Mạch được nghiên cứu và tối ưu hóa chuyên biệt cho hệ thống UAV của dự án AirAware, đảm bảo độ ổn định toàn vẹn tín hiệu, khả năng chịu dòng tải lớn và chống nhiễu EMI trong điều kiện bay thực tế.

## ⚙️ Thông số kỹ thuật (Hardware Specifications)
* **Vi điều khiển đích:** Radxa Zero 3W (Kết nối qua Header 40-pin).
* **Giao tiếp dữ liệu:** Tín hiệu USB 2.0 định tuyến vi sai (Differential Pair 90-ohm) kết nối module WiFi và Radxa.
* **Cấp nguồn (Power Management):** 
  * Nguồn vào: Pin UAV 3S-4S (11.1V - 16.8V).
  * Khối chuyển đổi (Buck Converter): Sử dụng IC TPS54302 (5V/2A) cung cấp năng lượng cho hệ thống WiFi.
  * Mạch bảo vệ: Tích hợp IC lý tưởng LTC4357 và MOSFET AO3400A chống cấp ngược cực.
* **Cấu trúc PCB:** Mạch 2 lớp (2-Layer PCB), độ dày 1.6mm. Phủ đồng GND (Polygon Pour) kín 2 mặt kết hợp Via Stitching tạo lồng Faraday chống nhiễu sóng.

## 🛠️ Nhật ký Thiết kế & Cập nhật (Changelog)

### Schematic (Cập nhật Sơ đồ nguyên lý)
* **[Fixed]** Sửa lỗi gộp nhầm net `$1N2075` ở khối nguồn Buck:
  * Tách tụ Output Bulk (C3, C4 22uF/10V) khỏi đường hồi tiếp (FB) và nối chuẩn về mặt phẳng GND để lọc nhiễu gợn sóng (Ripple).
  * Chuyển tụ Bootstrap (C5 100nF) mắc đúng giữa node SW và BOOT của IC TPS54302, đảm bảo mạch driver FET hoạt động.
* **[Optimized]** Cấu hình lại chuẩn giao tiếp:
  * Chập đúng các chân Dp1/Dp2 và Dn1/Dn2 trên cổng Type-C. Bỏ trống chân VBUS để loại trừ rủi ro xung đột nguồn với Radxa.
  * Hủy bỏ tụ debounce 10uF ở đường `WIFI_RST` để tín hiệu số bật/tắt module dứt khoát hơn.
  * Nối đúng sơ đồ bộ chia điện áp R1/R2 vào chân FB và điện trở kéo lên R45 vào `VIN_BAT`.

### PCB Layout (Định tuyến & Sắp xếp linh kiện)
* **[Rules Setup]** Áp dụng luật thiết kế (Design Rules) trong EasyEDA Pro:
  * Khoảng cách an toàn (Clearance) tiêu chuẩn: `0.254mm` (10 mil).
  * Khoảng cách chân linh kiện dán (SMD Pad clearance) ngoại lệ cho cổng Type-C: `0.152mm` (6 mil) để khắc phục cảnh báo DRC do mật độ chân siêu nhỏ.
* **[Routing Strategy]** 
  * Hủy Auto-router cho các đường tín hiệu quan trọng.
  * **Vẽ tay (Manual Routing)** mạng vi sai USB D+/D- ưu tiên lớp Top, đảm bảo tính toàn vẹn tín hiệu (Signal Integrity).
  * Quản lý phân lớp lưới điện (Net Class `PWR`): Không dùng dây tín hiệu mỏng, chuyển sang sử dụng vùng đồng cục bộ (Solid Region/Copper Pour) cho các node `VIN_BAT`, `+5V`, và `SW` để chịu tải dòng cao và tản nhiệt.
* **[Validated]** Vượt qua toàn bộ bài kiểm tra DRC (0 Errors, 0 Warnings).

## 🗂️ Quy trình xuất File Gia công (Manufacturing)
Để tiến hành sản xuất nguyên mẫu, sử dụng các chuẩn xuất file sau:
1. **Gerber Files:** (Định dạng RS-274X, nén .zip) dùng cho xưởng gia công PCB (JLCPCB).
2. **BOM & CPL Files:** (Định dạng .csv/.xlsx) phục vụ cho dịch vụ dán linh kiện tự động (SMT Assembly).
