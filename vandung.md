# BÀI TẬP: THIẾT KẾ GIẢI PHÁP PROMPTING ỨNG DỤNG THỰC TẾ
## KỸ THUẬT PROMPTING NÂNG CAO (LOGIC & CHUỖI)

## 1. Bối cảnh bài toán

**Tên dự án: TechShop – Tự động hóa xử lý đơn hàng.**

TechShop cần xây dựng hệ thống AI hỗ trợ tiếp nhận, kiểm tra, phân loại đơn hàng, tính doanh thu và tổng hợp báo cáo nhằm giảm thời gian xử lý và hạn chế sai sót.

Giải pháp sử dụng:
- **Chain-of-Thought:** Phân tích và xử lý vấn đề theo từng bước logic.
- **Prompt Chaining:** Chia quy trình thành nhiều prompt liên kết tuần tự.
- **Continue Prompting:** Tiếp tục công việc khi AI bị ngắt ngang.
- **Error Handling:** Phát hiện và xử lý dữ liệu lỗi.

## 2. Bước 1: Thiết kế giải pháp (Input/Output)

### 2.1. Input – Dữ liệu đầu vào

Thông tin cần cung cấp:

- Mã đơn hàng.
- Tên khách hàng.
- Tên sản phẩm.
- Số lượng sản phẩm.
- Đơn giá sản phẩm.

**Dữ liệu mẫu:**

```json
[
  {
    "order_id": "TS001",
    "customer": "Nguyen Van A",
    "product": "Laptop",
    "quantity": 2,
    "unit_price": 15000000
  },
  {
    "order_id": "TS002",
    "customer": "Tran Thi B",
    "product": "Dien thoai",
    "quantity": 1,
    "unit_price": 10000000
  },
  {
    "order_id": "TS003",
    "customer": "Le Van C",
    "product": "Tai nghe",
    "quantity": 3,
    "unit_price": 500000
  },
  {
    "order_id": "TS004",
    "customer": "Pham Van D",
    "product": "Ban phim",
    "quantity": -2,
    "unit_price": 300000
  }
]
```

### 2.2. Output – Kết quả mong muốn

Hệ thống cần trả về:

- Tổng số đơn hàng.
- Số đơn hàng hợp lệ và không hợp lệ.
- Doanh thu từng đơn hàng.
- Tổng doanh thu.
- Doanh thu theo nhóm sản phẩm.
- Danh sách lỗi dữ liệu.

Kết quả phải được xuất dưới dạng JSON hợp lệ, dễ kiểm tra và sử dụng.

## 3. Bước 2: Viết Prompt hoàn chỉnh

### 3.1. Prompt thực thi

```text
ROLE:
Bạn là chuyên viên phân tích dữ liệu của TechShop.

CONTEXT:
Cửa hàng cần tự động hóa quy trình xử lý đơn hàng.

TASK:
Thực hiện xử lý dữ liệu theo 5 bước:

1. Tiếp nhận và trích xuất thông tin đơn hàng.
2. Kiểm tra tính hợp lệ của dữ liệu.
3. Phân loại sản phẩm thành Laptop,
   Điện thoại, Phụ kiện, Thiết bị khác.
4. Tính doanh thu = Số lượng x Đơn giá.
5. Tổng hợp báo cáo cuối cùng.

RULES:
- Số lượng phải là số nguyên lớn hơn 0.
- Đơn giá phải là số không âm.
- Mã đơn hàng và tên sản phẩm không được trống.
- Phát hiện mã đơn hàng trùng lặp.
- Không tự suy đoán hoặc tạo dữ liệu còn thiếu.
- Đơn hàng lỗi không được tính doanh thu.
- Không để một đơn hàng lỗi làm dừng toàn bộ hệ thống.

ERROR HANDLING:
- Nếu dữ liệu bị thiếu, ghi nhận lỗi.
- Nếu sai định dạng, đánh dấu không hợp lệ.
- Nếu dữ liệu trùng lặp, đưa ra cảnh báo.
- Tiếp tục xử lý các đơn hàng hợp lệ.

PROMPT CHAINING:
Chia công việc thành 5 bước tuần tự.
Đầu ra của bước trước là đầu vào bước sau.
Kiểm tra kết quả trước khi chuyển bước.

CHAIN-OF-THOUGHT:
Phân tích logic nội bộ theo từng bước,
chỉ xuất kết quả và giải thích ngắn gọn cần thiết.

CONTINUE PROMPTING:
Nếu bị ngắt ngang, tiếp tục từ bước
chưa hoàn thành, giữ nguyên kết quả trước đó.
Không xử lý lại những bước đã hoàn tất.

OUTPUT:
Xuất JSON hợp lệ gồm:
- total_orders
- valid_orders
- invalid_orders
- total_revenue
- revenue_by_category
- errors
- status

Không thêm thông tin ngoài dữ liệu đầu vào.
```

## 4. Kết quả chạy thử

**Kết quả phản hồi của AI:**

```json
{
  "total_orders": 4,
  "valid_orders": 3,
  "invalid_orders": 1,
  "total_revenue": 41500000,
  "revenue_by_category": {
    "Laptop": 30000000,
    "Dien thoai": 10000000,
    "Phu kien": 1500000
  },
  "errors": [
    {
      "order_id": "TS004",
      "error": "So luong khong hop le: -2",
      "status": "invalid"
    }
  ],
  "status": "completed"
}
```

### 4.1. Đánh giá kết quả

- Tổng số đơn hàng: 4.
- Đơn hàng hợp lệ: 3.
- Đơn hàng không hợp lệ: 1.
- Tổng doanh thu: 41.500.000 VND.
- Đơn TS004 bị loại do số lượng âm.
- Các đơn hàng hợp lệ vẫn được xử lý bình thường.

## 5. Kết luận

Qua bài tập, em đã vận dụng Chain-of-Thought, Prompt Chaining và Continue Prompting để thiết kế giải pháp tự động hóa xử lý đơn hàng cho TechShop.

Giải pháp giúp chia nhỏ công việc, kiểm soát dữ liệu đầu vào, phát hiện lỗi và duy trì quy trình khi AI bị gián đoạn.

**Kết quả:** Xây dựng được quy trình xử lý đơn hàng có Input/Output rõ ràng, prompt chuyên nghiệp và kết quả JSON thống nhất, góp phần nâng cao độ chính xác và hiệu quả công việc.