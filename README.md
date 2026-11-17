# CNTT1-K-N-ng-Promting-v-i-AI-Session04_BTTH
# BÁO CÁO THỰC HÀNH TỔNG HỢP: KỸ THUẬT PROMPTING NÂNG CAO 

## 1. Báo cáo thiết kế giải pháp

* **Mô hình giải quyết:** Sử dụng **Prompt Chaining (Chuỗi Prompt liên kết)** kết hợp **Chain-of-Thought (CoT)** để chia nhỏ bài toán vận hành cửa hàng thành các bước độc lập, tránh tình trạng AI bị quá tải hoặc sinh lỗi "ảo giác".
* **Cơ chế bắt lỗi (Error Handling):**
* Xử lý dữ liệu dị biệt (thiếu thông tin, thiếu trường bắt buộc như địa chỉ, SĐT) bằng các điều kiện `[LỖI THIẾU THÔNG TIN]` để ngăn hệ thống tự bịa dữ liệu.
* Kiểm tra chéo tồn kho thực tế với số lượng đặt để chặn lỗi `[LỖI TỒN KHO]`, tự động ngắt các phần tính toán không hợp lệ mà không làm crash hệ thống.




## 2. Hệ thống Prompt thực thi tuần tự (Prompt Chaining)


Vai trò: Bạn là hệ thống xử lý và điều phối đơn hàng tự động. Hãy thực hiện tuần tự quy trình 3 bước sau đối với tin nhắn khách hàng: "{{USER_INPUT}}"

DỮ LIỆU KHO CỬA HÀNG:
- SP01 | Bàn phím cơ | Giá: 900.000đ | Tồn kho: 2 cái
- SP02 | Chuột gaming | Giá: 400.000đ | Tồn kho: 10 cái
- SP03 | Tai nghe chụp tai | Giá: 600.000đ | Tồn kho: 0 cái (Hết hàng)
- Vận chuyển: Đơn >= 1.000.000đ miễn phí ship; đơn < 1.000.000đ tính phí ship 30.000đ.

---
BƯỚC 1: TRÍCH XUẤT VÀ BẮT LỖI DỮ LIỆU
- Đọc tin nhắn, trích xuất: Tên khách, SĐT, Địa chỉ, Sản phẩm & Số lượng.
- Kiểm tra điều kiện lỗi: 
  + Nếu thiếu Tên, SĐT hoặc Địa chỉ -> Ghi rõ: [LỖI THIẾU THÔNG TIN]. Tuyệt đối không tự suy diễn.
  + Nếu số lượng đặt vượt tồn kho hoặc sản phẩm hết hàng -> Ghi rõ: [LỖI TỒN KHO].

BƯỚC 2: SUY LUẬN LOGIC TÍNH TOÁN (Chain-of-Thought)
- Chỉ tính tiền cho sản phẩm hợp lệ và còn hàng trong kho. Bỏ qua sản phẩm lỗi tồn kho.
- Diễn giải các bước tính: Thành tiền tạm tính -> Phí vận chuyển -> Tổng thanh toán cuối cùng.

BƯỚC 3: KẾT XUẤT KẾT QUẢ
Trình bày kết quả duy nhất theo khung mẫu chuẩn sau:
=== PHIẾU ĐƠN HÀNG ===
- Trạng thái: [Hợp lệ / Cần bổ sung / Lỗi kho]
- Cảnh báo lỗi: [Nội dung lỗi ở Bước 1, nếu không có ghi "Không"]
- Khách hàng: [Tên] - [SĐT] - [Địa chỉ]
- Chi tiết lập luận & Tính tiền: [Tóm tắt ngắn gọn các bước CoT]
- Tổng thanh toán: [Số tiền] VNĐ


## 3. Kết quả chạy thử thực tế (Test Run)

* **Dữ liệu đầu vào (`USER_INPUT`):**
 *"Shop ship cho mình 2 cái bàn phím cơ (SP01) và 1 cái tai nghe chụp tai (SP03) nhé. Mình tên Lan, sđt 0911223344."*


* **Kết quả phản hồi từ mô hình:**

=== PHIẾU ĐƠN HÀNG ===
- Trạng thái: Cần bổ sung & Lỗi kho
- Cảnh báo lỗi: 
  + [LỖI THIẾU THÔNG TIN]: Khách hàng chưa cung cấp địa chỉ giao hàng.
  + [LỖI TỒN KHO]: Sản phẩm SP03 (Tai nghe chụp tai) đã hết hàng trong kho (tồn kho 0).
  + [LỖI TỒN KHO]: Sản phẩm SP01 (Bàn phím cơ) khách đặt 2 cái vừa khớp với số lượng tồn kho (2 cái), cần lưu ý kho không còn dự phòng.
- Khách hàng: Lan - 0911223344 - Chưa cung cấp
- Chi tiết lập luận & Tính tiền:
  + Bước 2.1: Tính tiền cho sản phẩm hợp lệ SP01: 2 x 900.000đ = 1.800.000đ. (Ngắt phần tính toán SP03 do hết hàng).
  + Bước 2.2: Tổng tiền hàng tạm tính = 1.800.000đ.
  + Bước 2.3: Vì 1.800.000đ >= 1.000.000đ nên đơn hàng được Miễn phí vận chuyển (0đ).
  + Bước 2.4: Tổng thanh toán cuối cùng = 1.800.000đ + 0đ = 1.800.000đ.
- Tổng thanh toán: 1.800.000 VNĐ (Tạm tính cho 2 bàn phím cơ, chờ khách bổ sung địa chỉ)
