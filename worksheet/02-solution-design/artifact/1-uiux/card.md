# Card — Lớp UI/UX

**Layer:** L1 — Giao diện người dùng
**Rủi ro giải quyết:** Hallucination về deadline/học bổng/hồ sơ tuyển sinh (C-01, C-02)

---

## Vấn đề cần giải quyết

User tin tưởng hoàn toàn vào chatbot vì nằm trên website chính thức của trường. Giao diện hiện tại không có tín hiệu nào cho thấy:
- Câu trả lời đến từ nguồn nào và ngày nào được cập nhật
- Câu trả lời có phải là xác nhận chính thức không
- User nên làm gì nếu cần xác nhận chắc chắn hơn

---

## Giải pháp UI/UX — 5 thành phần

### 1. Source Badge (Nhãn nguồn)
Mỗi câu trả lời chứa thông tin cụ thể (deadline, học bổng, học phí, điều kiện hồ sơ) phải kèm nhãn:
```
[Nguồn: Thông báo Tuyển sinh 2025 | Cập nhật: 15/01/2025]
```
Nếu AI không có nguồn rõ → không được hiển thị số liệu → thay bằng fallback message.

### 2. Warning Banner (Cảnh báo chủ đề nhạy cảm)
Khi topic là deadline, học bổng, điều kiện xét tuyển → hiển thị banner màu vàng phía trên câu trả lời:
```
⚠️ Thông tin này mang tính tham khảo. Để xác nhận chính thức, vui lòng kiểm tra trang tuyển sinh hoặc liên hệ tư vấn viên.
```

### 3. Confidence Indicator (Chỉ số độ tin cậy)
Mỗi câu trả lời có một trong ba trạng thái hiển thị:
- **Đã xác nhận** (màu xanh) — có nguồn chính thức và còn hạn
- **Chưa xác nhận** (màu vàng) — không có nguồn chính thức hoặc nguồn cũ
- **Cần xác nhận từ tư vấn viên** (màu cam) — case cá nhân / ngoại lệ

### 4. Persistent Counselor Button (Nút tư vấn viên cố định)
Khi topic là deadline/học bổng/hồ sơ → nút "Nói chuyện với tư vấn viên" luôn hiển thị dưới câu trả lời, không bị ẩn:
```
[💬 Nói chuyện với tư vấn viên]   [🔗 Xem trang tuyển sinh chính thức]
```

### 5. Persistent Disclaimer (Tuyên bố miễn trách nhiệm cố định)
Dưới chatbox, luôn hiển thị dòng chữ nhỏ:
```
Chatbot AI hỗ trợ thông tin tham khảo — không thay thế xác nhận chính thức từ Phòng Tuyển sinh.
Hotline: [số điện thoại] | Email: [email tuyển sinh] | [Trang tuyển sinh]
```

---

## Tại sao giải pháp này hoạt động

| Thành phần | Root cause được khắc phục |
|---|---|
| Source Badge | User biết thông tin đến từ đâu và cập nhật ngày nào → không mặc định tin như cam kết chính thức |
| Warning Banner | Chủ động cảnh báo trước khi user đưa ra quyết định sai |
| Confidence Indicator | Phân biệt rõ mức độ chắc chắn — "xác nhận" vs "tham khảo" |
| Counselor Button | Luôn có đường thoát rõ ràng ra khỏi chatbot khi cần xác nhận thật |
| Disclaimer | Đặt kỳ vọng đúng từ đầu: chatbot là hỗ trợ, không phải kênh chính thức |

---

## Giới hạn của lớp này

- Không ngăn AI đưa thông tin sai nếu nguồn trong knowledge base đã sai từ đầu (cần L3 xử lý).
- User có thể bỏ qua cảnh báo nếu không đọc kỹ (cần L2 đảm bảo AI cũng tự từ chối).
- Chỉ hoạt động tốt khi L2 (Prompt) cung cấp metadata nguồn chính xác để hiển thị.
