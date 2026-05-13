# Card — Lớp Kiến trúc

**Layer:** L3 — Kiến trúc dữ liệu và hệ thống
**Rủi ro giải quyết:** Stale data (C-06), Input conflict (C-05), Scaled hallucination (C-08), Escalation failure (C-03)

---

## Vấn đề cần giải quyết

Ngay cả khi có guardrail prompt tốt (L2), model vẫn có thể:
- Truy xuất tài liệu đã hết hạn từ knowledge base (stale data)
- Nhận hai tài liệu mâu thuẫn về cùng một deadline
- Không có cơ chế tự động route escalation sang counselor thật
- Lỗi thông tin không được ghi lại để admissions team phát hiện và sửa

Lớp Architecture xử lý các vấn đề này ở tầng hạ tầng — trước khi model nhận được dữ liệu.

---

## Giải pháp — 5 thành phần kiến trúc

### A1 — Document Metadata Schema (Metadata bắt buộc)

Mỗi tài liệu được nạp vào knowledge base phải có các trường metadata:

```json
{
  "doc_id": "scholarship-cs-2025",
  "title": "Thông báo Học bổng Ngành KHMT 2025",
  "document_type": "scholarship",
  "source_url": "https://admissions.truong.edu.vn/hoc-bong-khmt-2025",
  "effective_date": "2025-01-15",
  "expiry_date": "2025-07-30",
  "last_verified": "2025-01-15",
  "sensitivity": "high",
  "topics": ["deadline", "scholarship", "admission-requirement"]
}
```

Tài liệu không có đầy đủ metadata → không được index vào RAG.

### A2 — Staleness Detector (Phát hiện tài liệu hết hạn)

Một job tự động chạy hàng ngày:
- So sánh `expiry_date` và `last_verified` của mỗi tài liệu với ngày hiện tại
- Tài liệu có `expiry_date < today` hoặc `last_verified` > 30 ngày và `sensitivity = "high"` → bị đánh dấu `status: "stale"`
- Tài liệu `stale` bị loại khỏi RAG retrieval pool → model không thể truy xuất

### A3 — Fallback Handler (Xử lý khi không có tài liệu)

Khi RAG không tìm được tài liệu phù hợp (hoặc tài liệu bị `stale`):
- Thay vì để model tự suy đoán → hệ thống trả về **Fallback Response Template** chuẩn
- Template bao gồm: lý do không có thông tin + link trang chính thức + contact counselor
- Template được cập nhật thủ công bởi admissions team — không do AI tạo ra

### A4 — Counselor Routing API (Route escalation tự động)

Khi L2 (Prompt) trigger escalation hoặc user bấm nút "Nói chuyện với tư vấn viên":
- API tạo một session với counselor (Livechat / ticket / email thread)
- Đính kèm toàn bộ context hội thoại hiện tại vào session → counselor biết ngay user đang hỏi gì
- Ghi log: `escalation_reason`, `user_query`, `bot_response`, `timestamp`
- Nếu counselor offline → auto-reply: "Tư vấn viên sẽ phản hồi trong vòng [X giờ]. Hotline khẩn: [số]."

### A5 — Audit Log & Review Pipeline (Ghi log và kiểm tra định kỳ)

Mọi câu hỏi có topic `["deadline", "scholarship", "tuition", "admission-requirement"]` được ghi:

```json
{
  "timestamp": "2026-01-15T23:30:00",
  "user_query": "Deadline học bổng 2026 là ngày nào?",
  "retrieved_docs": ["scholarship-cs-2025"],
  "doc_status": "stale",
  "bot_response_type": "fallback",
  "escalated": false
}
```

Admissions team nhận báo cáo hàng tuần:
- Top 10 câu hỏi không tìm được tài liệu → update knowledge base
- Số case escalation → đánh giá tải counselor
- Số lần tài liệu `stale` được truy xuất → trigger review tài liệu

---

## Tại sao giải pháp này hoạt động

| Thành phần | Root cause được khắc phục |
|---|---|
| A1 — Metadata Schema | Tài liệu không rõ nguồn/ngày không được vào hệ thống → giảm stale data từ đầu |
| A2 — Staleness Detector | Tài liệu hết hạn bị loại trước khi model nhận → giảm hallucination từ nguồn cũ |
| A3 — Fallback Handler | Thay thế model suy đoán bằng fallback được kiểm soát → consistent refusal |
| A4 — Counselor Routing API | Escalation tự động có context đầy đủ → counselor xử lý được ngay |
| A5 — Audit Log | Admissions team nhìn thấy lỗi → cập nhật knowledge base và guardrail kịp thời |

---

## Giới hạn của lớp này

- Cần đầu tư vào engineering để tích hợp metadata pipeline và staleness detector.
- Fallback template cần được admissions team cập nhật thường xuyên — nếu không thì link và contact cũng có thể lỗi thời.
- Counselor Routing API cần tích hợp với hệ thống CRM/Livechat của trường.
- Audit log chỉ có giá trị nếu admissions team thực sự đọc và hành động theo báo cáo.
