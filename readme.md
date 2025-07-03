
# 📄 Thiết kế API và Mô hình Dữ liệu cho Dịch vụ Rút gọn URL

## 🎯 Mục tiêu
Thiết kế một dịch vụ rút gọn URL đơn giản giống như Bit.ly, cho phép người dùng ẩn danh gửi URL dài và nhận lại URL ngắn. Hệ thống hoạt động không yêu cầu đăng nhập và cho phép tạo nhiều mã rút gọn cho cùng một URL gốc.

---

## 📌 1. API Specification

### 🔹 `POST /api/urls` – Tạo URL ngắn mới

- **Mô tả:** Tạo một mã rút gọn cho URL gốc được cung cấp.
- **Request:**
```json
{
  "original_url": "https://example.com/some/long/path"
}
```
- **Response (201 Created):**
```json
{
  "short_code": "aB3dEf",
  "short_url": "https://short.ly/aB3dEf",
  "original_url": "https://example.com/some/long/path",
  "created_at": "2025-07-03T10:00:00Z"
}
```
- **Status codes:**
  - `201 Created`: Tạo thành công
  - `400 Bad Request`: Thiếu hoặc sai định dạng URL

---

### 🔹 `GET /{short_code}` – Chuyển hướng đến URL gốc

- **Mô tả:** Truy xuất URL gốc và chuyển hướng đến đó.
- **Request:** Không cần body.
- **Response:**
  - `302 Found`: Chuyển hướng đến `original_url`
- **Status codes:**
  - `302 Found`: Chuyển hướng thành công
  - `404 Not Found`: Không tìm thấy `short_code`

---

## 🗃️ 2. Database Schema (SQL - PascalCase)

Sử dụng cơ sở dữ liệu quan hệ (PostgreSQL hoặc MySQL). Một bảng duy nhất: `ShortUrls`

| Tên cột       | Kiểu dữ liệu     | Ghi chú |
|---------------|------------------|--------|
| Id            | SERIAL / BIGSERIAL | Khóa chính, dùng để encode ra short_code |
| OriginalUrl   | TEXT              | URL gốc |
| ShortCode     | VARCHAR(10)       | Mã rút gọn (sinh từ Id bằng thuật toán Base62) |
| CreatedAt     | TIMESTAMP         | Ngày tạo |
| VisitCount    | INTEGER DEFAULT 0 | Số lượt truy cập |

### 🔍 Indexes đề xuất:
- `ShortCode` – Unique Index (để truy vấn nhanh khi redirect)
- `CreatedAt` – Index (cho truy vấn thống kê)
- `OriginalUrl` – Có thể tạo Index nếu cần thống kê hoặc kiểm tra trùng URL

---

## 🔐 3. Thuật toán tạo ShortCode (Theo mô hình Bitly)

### 🎯 Mục tiêu:
- Sinh mã rút gọn ngắn gọn, không trùng lặp, dễ redirect và không cần người dùng đăng nhập.

### 📌 Các bước thực hiện:

1. **Tạo bản ghi mới:** Insert `OriginalUrl` vào bảng `ShortUrls` → Hệ quản trị sẽ sinh `Id` tự tăng.
2. **Encode:** Dùng thuật toán Base62 để chuyển `Id` thành `ShortCode`.
3. **Cập nhật:** Update lại bản ghi với giá trị `ShortCode` tương ứng.
4. **Trả kết quả:** Trả về URL rút gọn có dạng `https://short.ly/{ShortCode}`.

### ✏️ Ví dụ SQL:

```sql
-- Bước 1: Tạo bản ghi và lấy Id
INSERT INTO ShortUrls (OriginalUrl)
VALUES ('https://example.com/very/long/url')
RETURNING Id;

-- Giả sử Id = 125, thì ứng dụng sẽ encode base62(125) => 'cb'

-- Bước 2: Cập nhật lại ShortCode
UPDATE ShortUrls SET ShortCode = 'cb' WHERE Id = 125;
```

---

## 🧠 Hành vi hệ thống

| Tình huống | Kết quả |
|-----------|---------|
| Gửi lại cùng `OriginalUrl` | Tạo mới `ShortCode` khác |
| Truy cập `/aB3dEf` | Chuyển hướng đến URL gốc |
| `ShortCode` không tồn tại | Trả `404 Not Found` |

---

## ✅ Kết luận

Hệ thống rút gọn URL theo mô hình Bitly, ẩn danh, hoạt động hiệu quả với cơ chế sinh `ShortCode` từ `Id` tăng tự động. Dữ liệu được tối ưu để mở rộng trong tương lai, đồng thời cho phép nhiều mã ngắn cho cùng một URL gốc.



### 🎯 Lịch sử sử dụng Chat GPT:
https://chatgpt.com/share/686658af-d16c-800c-a201-0a060bebe5bc

