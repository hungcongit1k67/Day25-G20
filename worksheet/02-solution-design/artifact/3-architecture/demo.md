# Demo — Lớp Kiến trúc

**Sơ đồ luồng dữ liệu và ví dụ hoạt động của từng thành phần**

---

## Sơ đồ tổng thể: Luồng xử lý một câu hỏi về deadline

```
USER HỎI: "Deadline học bổng KHMT 2026 là ngày nào?"
                    │
                    ▼
        ┌───────────────────────┐
        │  QUERY CLASSIFIER     │
        │  Topic: deadline      │
        │  Sensitivity: high    │
        └──────────┬────────────┘
                   │
                   ▼
        ┌───────────────────────┐
        │  RAG RETRIEVAL        │
        │  Tìm tài liệu về     │
        │  "học bổng KHMT 2026"│
        └──────────┬────────────┘
                   │
          ┌────────┴────────┐
          │                 │
    Tìm được            Không tìm được
    tài liệu             tài liệu
          │                 │
          ▼                 ▼
  ┌───────────────┐  ┌──────────────────────┐
  │ A2: Staleness │  │ A3: Fallback Handler │
  │ Detector      │  │ Trả về template từ  │
  │               │  │ chối + link official │
  │ Kiểm tra:     │  │ + Counselor contact  │
  │ expiry_date   │  └──────────────────────┘
  │ last_verified │
  └───────┬───────┘
          │
  ┌───────┴───────┐
  │               │
Còn hạn       Hết hạn / stale
  │               │
  ▼               ▼
Model nhận    A3: Fallback
tài liệu +    Handler kích
metadata      hoạt
  │               │
  ▼               │
L2 Prompt         │
Guardrail         │
(cite nguồn,      │
 escalation)      │
  │               │
  └───────┬───────┘
          │
          ▼
  ┌───────────────────────┐
  │  L1 UI/UX             │
  │  Hiển thị response   │
  │  + Badge nguồn        │
  │  + Warning Banner     │
  │  + Counselor Button   │
  └───────────────────────┘
          │
          ▼
  ┌───────────────────────┐
  │  A5: Audit Log        │
  │  Ghi log query,       │
  │  doc_status,          │
  │  response_type,       │
  │  escalated            │
  └───────────────────────┘
```

---

## Ví dụ A1 — Document Metadata

**Tài liệu HỢP LỆ (sẽ được index):**
```json
{
  "doc_id": "scholarship-cs-2025-v2",
  "title": "Thông báo Học bổng Ngành KHMT - Kỳ tuyển sinh 2025",
  "document_type": "scholarship",
  "source_url": "https://admissions.truong.edu.vn/hoc-bong-khmt-2025",
  "effective_date": "2025-03-01",
  "expiry_date": "2025-08-31",
  "last_verified": "2025-06-15",
  "sensitivity": "high",
  "topics": ["deadline", "scholarship", "ielts-requirement", "application"]
}
```

**Tài liệu KHÔNG HỢP LỆ (bị từ chối index):**
```json
{
  "doc_id": "old-faq-2023",
  "title": "FAQ Tuyển sinh 2023",
  "document_type": "general",
  "source_url": null,
  "expiry_date": null,       ← THIẾU expiry_date
  "last_verified": null,     ← THIẾU last_verified
  "sensitivity": "high"
}
→ BỊ TỪ CHỐI — không đủ metadata bắt buộc
```

---

## Ví dụ A2 — Staleness Detector (chạy hàng ngày lúc 00:00)

```
Ngày kiểm tra: 2026-01-15

Tài liệu: scholarship-cs-2025-v2
  expiry_date: 2025-08-31  < 2026-01-15  → HẾT HẠN
  → Đánh dấu status: "stale"
  → Loại khỏi RAG pool
  → Gửi alert: "Cần cập nhật tài liệu học bổng KHMT cho 2026"

Tài liệu: general-admission-guide-2026
  expiry_date: 2026-07-31  > 2026-01-15  → CÒN HẠN
  last_verified: 2026-01-10  (5 ngày trước)  → OK
  → Giữ trong RAG pool
```

---

## Ví dụ A3 — Fallback Handler response

**Khi không tìm được tài liệu hợp lệ:**

```
[HỆ THỐNG — Fallback Template v2.1 | Cập nhật: 2026-01-10]

Mình chưa có thông tin chính thức và cập nhật về học bổng ngành KHMT
năm 2026 trong hệ thống.

Để xác nhận chính xác:
→ Xem thông báo học bổng: admissions.truong.edu.vn/hoc-bong
→ Xem deadline tuyển sinh: admissions.truong.edu.vn/deadline
→ Liên hệ tư vấn viên: tuyen-sinh@truong.edu.vn
→ Hotline: 1900-xxxx (Thứ 2–6, 8:00–17:00)

Mình không thể đoán hoặc ước lượng thông tin này.

[💬 Kết nối với tư vấn viên ngay]
```

---

## Ví dụ A4 — Counselor Routing khi escalation

**Khi user bấm "Kết nối với tư vấn viên" hoặc prompt trigger escalation:**

```
API call: POST /api/escalate
{
  "session_id": "sess-20260115-0023",
  "escalation_reason": "user_request + high_stakes_topic",
  "conversation_context": [
    {"role": "user", "content": "Nhà em khó khăn..."},
    {"role": "bot", "content": "Đây là trường hợp cần..."}
  ],
  "user_query_summary": "Thiếu IELTS + thư giới thiệu, hỏi có được xét HB không",
  "timestamp": "2026-01-15T23:31:00"
}

→ Hệ thống tạo ticket #TK-2026-0042
→ Gán cho counselor trực (nếu có)
→ Nếu offline: auto-reply với SLA: "Phản hồi trong 2 giờ làm việc"
→ Counselor nhận email với đầy đủ context
```

---

## Ví dụ A5 — Audit Log report (hàng tuần)

```
BÁO CÁO TUẦN: 06/01/2026 – 12/01/2026

Tổng câu hỏi high-sensitivity: 342
  → Có tài liệu còn hạn: 198 (58%)
  → Fallback (không có / stale): 144 (42%)  ← CẦN CHÚ Ý

Top 5 câu hỏi không có tài liệu:
  1. "Deadline học bổng KHMT 2026" — 67 lần
  2. "Học phí 2026-2027" — 45 lần
  3. "Điều kiện học bổng tài năng 2026" — 38 lần
  4. "Deadline nộp hồ sơ ngành Kinh tế 2026" — 29 lần
  5. "Giấy tờ hồ sơ bằng THPT nước ngoài" — 21 lần

Tổng escalation: 28 case
  → Counselor đã xử lý: 25
  → Chờ xử lý: 3

ĐỀ XUẤT: Cần cập nhật tài liệu cho 5 topic trên trước 20/01/2026.
```

---

## Mapping thành phần → test case

| Test case | Thành phần A xử lý |
|---|---|
| F-01: Deadline 2026 chưa công bố | A2 (stale/không có tài liệu) → A3 (fallback) |
| F-03: Nguồn mâu thuẫn | A1 (metadata conflict detection) + A2 (loại tài liệu cũ hơn) |
| F-07: Nộp muộn xin ngoại lệ | A4 (escalation route + context đính kèm) |
| F-09: Câu hỏi lặp 3 lần → kết quả khác nhau | A3 (fallback nhất quán thay vì model suy đoán) |
| F-11: Hỏi lúc 11:30 PM sắp deadline | A4 (auto-reply SLA + hotline khẩn) |
| C-08: Lỗi scaled (cache sai mùa cao điểm) | A2 (stale tập trung) + A5 (audit detect spike) |
