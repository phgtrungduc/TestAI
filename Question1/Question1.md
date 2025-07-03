
# Thiết kế API và Mô hình Dữ liệu cho Dịch vụ Rút gọn URL

## 📌 Mô tả dự án

Dịch vụ rút gọn URL giúp chuyển đổi một URL dài thành một mã rút gọn ngắn, dễ nhớ. Khi người dùng truy cập vào mã rút gọn, hệ thống sẽ chuyển hướng về URL gốc.

Ví dụ:
- URL gốc: `https://example.com/articles/how-to-build-a-url-shortener`
- URL rút gọn: `https://short.ly/abc123`

---

## 🧩 1. API Specification

### 🔹 Endpoint 1: POST `/api/urls`

#### ➤ Mô tả:
Tạo một URL rút gọn từ một URL gốc.

#### ➤ Request:
- **Phương thức:** `POST`
- **Content-Type:** `application/json`
- **Body:**
```json
{
  "original_url": "https://example.com/very-long-url"
}
```

#### ➤ Response:
- **Status:** `201 Created`
- **Body:**
```json
{
  "short_code": "abc123",
  "short_url": "https://short.ly/abc123"
}
```

#### ➤ Các mã trạng thái:
- `201 Created`: Tạo thành công
- `400 Bad Request`: Thiếu hoặc sai định dạng `original_url`

---

### 🔹 Endpoint 2: GET `/{short_code}`

#### ➤ Mô tả:
Chuyển hướng đến `original_url` tương ứng với `short_code`.

#### ➤ Request:
- **Phương thức:** `GET`
- **URL:** `https://short.ly/abc123`

#### ➤ Response:
- **Status:** `302 Found`
- **Redirect To:** `https://example.com/very-long-url`

#### ➤ Các mã trạng thái:
- `302 Found`: Chuyển hướng thành công
- `404 Not Found`: `short_code` không tồn tại

---

## 🗃️ 2. Database Schema

### 🔸 Cấu trúc bảng (SQL-based schema)

| Trường        | Kiểu dữ liệu | Mô tả                              |
|---------------|--------------|------------------------------------|
| `id`          | SERIAL / UUID| Khóa chính                         |
| `original_url`| TEXT         | URL gốc dài                        |
| `short_code`  | VARCHAR(10)  | Mã rút gọn, duy nhất               |
| `created_at`  | TIMESTAMP    | Ngày tạo bản ghi                   |
| `visit_count` | INTEGER      | (Tuỳ chọn) Đếm số lần truy cập     |

### 🔸 Ràng buộc & chỉ mục (Indexing)

| Trường         | Index kiểu gì     | Ghi chú                                  |
|----------------|-------------------|------------------------------------------|
| `short_code`   | `UNIQUE INDEX`    | ⚠️ Bắt buộc — để redirect nhanh chóng    |
| `original_url` | `INDEX` (tuỳ chọn)| ✔ Nếu muốn tránh tạo trùng URL           |
| `created_at`   | `INDEX` (tuỳ chọn)| ✔ Nếu có dashboard, lọc hoặc xoá theo thời gian |

---

## 🧠 3. Gợi ý kỹ thuật

- **Sinh short_code**: dùng `base62`, `hashids`, hoặc random 6 ký tự alphanumeric (`A-Za-z0-9`)
- **Ngôn ngữ gợi ý**: Python (Flask, FastAPI), Node.js (Express)
- **Cơ sở dữ liệu gợi ý**: PostgreSQL, SQLite (SQL); MongoDB (NoSQL)
- **Cân nhắc mở rộng**:
  - Tạo `custom_alias`
  - Thời gian hết hạn URL (`expires_at`)
  - Bảo vệ chống spam/bot (captcha, rate limit)

---

## ✅ Ví dụ minh họa

### Tạo URL rút gọn:

**Request:**
```http
POST /api/urls
Content-Type: application/json

{
  "original_url": "https://youtube.com/watch?v=abcxyz123"
}
```

**Response:**
```json
{
  "short_code": "yt89pQ",
  "short_url": "https://short.ly/yt89pQ"
}
```

---

### Truy cập short_code:

**Request:**
```http
GET /yt89pQ
```

**Response:**
- Status: `302 Found`
- Location: `https://youtube.com/watch?v=abcxyz123`

---

## 🔒 Bảo mật & hiệu năng

- **Bảo mật**:
  - Validate input (URL hợp lệ)
  - Tránh SQL Injection (dùng ORM hoặc query param)
  - Hạn chế gửi quá nhiều request (rate limit)

- **Hiệu năng**:
  - Index `short_code` là bắt buộc
  - Index `created_at`, `original_url` nếu có nhu cầu truy vấn thêm
  - Cache kết quả truy vấn `short_code`
  - Scale: tách đọc/ghi DB nếu lớn

---

## 📚 Tài liệu tham khảo

- [Bitly Developer Docs](https://dev.bitly.com/)
- [Base62 Encoding](https://en.wikipedia.org/wiki/Base62)
- [FastAPI Docs](https://fastapi.tiangolo.com/)
- [MongoDB Data Model Design](https://www.mongodb.com/docs/manual/core/data-modeling-introduction/)

---

## 🧑‍💻 Người thực hiện

- Họ tên: (Điền họ tên bạn)
- Ngày thực hiện: (Điền ngày làm bài)
