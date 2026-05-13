# Card — Lớp Prompt

**Layer:** L2 — Chỉ dẫn AI (System Prompt Guardrails)
**Rủi ro giải quyết:** Hallucination (C-01, C-02), Sycophancy (C-04), Escalation failure (C-03)

---

## Vấn đề cần giải quyết

Model AI có xu hướng:
- Sinh câu trả lời chắc chắn dù không có nguồn xác minh (confidence ≠ accuracy)
- Chiều lòng user khi bị áp lực ("cứ nói đại đi", "em cần nghe điều đó")
- Không tự nhận ra khi nào cần chuyển sang counselor

Lớp UI chỉ cảnh báo người dùng sau khi AI đã trả lời. Lớp Prompt phải ngăn AI đưa ra câu trả lời sai ngay từ đầu.

---

## Giải pháp — 4 guardrail trong System Prompt

### G1 — Citation mandate (Bắt buộc trích nguồn)

Mọi câu trả lời có thông tin cụ thể về deadline, học bổng, học phí, điều kiện hồ sơ **phải** kèm:
```
[Nguồn: <tên tài liệu> | Ngày cập nhật: <ngày>]
```
Nếu không có tài liệu có ngày cụ thể → **không được đưa số liệu** → dùng Refusal template (G2).

### G2 — Refusal template (Từ chối chuẩn khi thiếu nguồn)

Khi không có nguồn chính thức cho câu hỏi:
```
Mình chưa có thông tin chính thức và cập nhật về [chủ đề] cho [năm/học kỳ] trong hệ thống.
Để xác nhận chính xác, bạn nên:
→ Kiểm tra tại: [link trang chính thức liên quan]
→ Hoặc liên hệ tư vấn viên: [kênh liên hệ]
Mình không thể đoán hoặc ước lượng thông tin này vì có thể làm ảnh hưởng đến quyết định của bạn.
```

### G3 — Pressure resistance (Từ chối áp lực đoán)

Khi user nói: "nói đại đi", "ước lượng thôi", "không cần chắc 100%", "cứ cho tôi ngày gần đúng" → AI **phải** từ chối và giải thích:
```
Mình hiểu bạn đang cần quyết định nhanh. Nhưng nếu mình đoán deadline và thông tin đó sai,
bạn có thể lỡ cơ hội học bổng hoặc nộp hồ sơ sai thời điểm — điều mình không muốn xảy ra.
Cách an toàn nhất ngay bây giờ là: [link trang official] hoặc gọi hotline [số].
```

### G4 — Escalation trigger (Kích hoạt chuyển counselor)

AI **phải** chủ động offer counselor khi câu hỏi thuộc một trong các case:
- Deadline của năm tuyển sinh hiện tại (topic: deadline + năm cụ thể)
- Điều kiện học bổng có yếu tố cá nhân (thiếu điều kiện, hoàn cảnh đặc biệt)
- Nộp hồ sơ muộn / xin ngoại lệ
- Cần xác nhận có/không đủ điều kiện cho một học bổng cụ thể

Câu offer chuẩn:
```
Đây là trường hợp cần xác nhận từ tư vấn viên. Mình có thể kết nối bạn với tư vấn viên ngay bây giờ không?
[💬 Kết nối với tư vấn viên]
```

---

## Tại sao guardrail này hoạt động

| Guardrail | Root cause được khắc phục |
|---|---|
| G1 — Citation mandate | Buộc model không thể đưa số liệu nếu không có nguồn → giảm hallucination |
| G2 — Refusal template | Tạo hành vi từ chối nhất quán thay vì để model tự suy đoán cách xử lý |
| G3 — Pressure resistance | Giữ boundary ngay cả khi user cố tình tạo áp lực → chống sycophancy |
| G4 — Escalation trigger | Tự động nhận diện case cần người thật → giảm escalation failure |

---

## Giới hạn của lớp này

- Prompt guardrail có thể bị bypass trong các cuộc hội thoại dài (context dilution).
- Hiệu quả phụ thuộc vào chất lượng tài liệu trong knowledge base — nếu tài liệu sai thì citation cũng sai (cần L3 xử lý).
- Cần test và tinh chỉnh thường xuyên vì model có thể "sáng tạo" cách diễn đạt vẫn vi phạm spirit của guardrail.
- Không thể kiểm soát 100% khi model được fine-tune thêm hoặc có system prompt override.
