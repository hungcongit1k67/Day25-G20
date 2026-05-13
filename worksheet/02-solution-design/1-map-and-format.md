# 1-map-and-format — Bản đồ Thiết kế Giải pháp

**Bài 2 — Solution Design**
**Nhóm G20 | Track 01 — Chatbot tư vấn tuyển sinh đại học**

---

## Rủi ro được chọn để thiết kế giải pháp

**C-01 / C-02 (Risk Score 25/25) — Hallucination về deadline và học bổng**

> Khi user hỏi deadline, điều kiện học bổng, hồ sơ hoặc học phí trong một năm tuyển sinh cụ thể, AI có xu hướng bịa câu trả lời chắc chắn thay vì thừa nhận thiếu nguồn chính thức và chuyển sang kênh xác nhận — gây rủi ro lỡ deadline, mất cơ hội học bổng và ra quyết định tài chính sai cho học sinh/phụ huynh.

**Lý do chọn:** Risk Score cao nhất (25/25), hậu quả thực tế trực tiếp và không thể hoàn tác (lỡ deadline học bổng), lan rộng khi chatbot phục vụ hàng nghìn user mùa tuyển sinh. Giải pháp cho rủi ro này cũng phủ một phần C-04 (Sycophancy) và C-06 (Stale data).

---

## Root Cause Analysis

| Lớp | Root cause |
|---|---|
| **Input / Data** | Knowledge base chứa tài liệu chưa cập nhật, không có metadata ngày hết hạn; PDF scan lỗi không được index đúng; hai tài liệu mâu thuẫn cùng tồn tại trong RAG |
| **Model** | Model có xu hướng sinh câu trả lời chắc chắn dù không có nguồn (confidence ≠ accuracy); thiếu guardrail buộc trích nguồn hoặc từ chối khi không có dữ liệu |
| **UI/UX** | Giao diện không hiển thị nguồn, ngày cập nhật, mức độ chắc chắn; không có cảnh báo "đây không phải xác nhận chính thức"; không có nút chuyển sang counselor dễ thấy |

---

## Ba lớp giải pháp

### Tổng quan

| Lớp | Tên | Mục tiêu chính | File |
|---|---|---|---|
| **L1** | UI/UX | Giúp user nhận biết giới hạn của AI và tìm được kênh xác nhận chính thức | `artifact/1-uiux/` |
| **L2** | Prompt | Buộc AI trích nguồn, từ chối khi thiếu dữ liệu, escalate khi cần | `artifact/2-prompt/` |
| **L3** | Architecture | Đảm bảo knowledge base luôn có dữ liệu đúng, hết hạn được flagged, escalation được route tự động | `artifact/3-architecture/` |

---

### L1 — UI/UX (Giao diện)

**Vấn đề cần giải quyết:** User đặt tin tưởng cao vào chatbot vì nằm trên website chính thức. Không có gì cho user thấy rằng câu trả lời có thể chưa được xác nhận chính thức.

**Giải pháp:**
- Badge "Nguồn: [tên tài liệu] — Cập nhật [ngày]" dưới mỗi câu trả lời có số liệu cụ thể
- Banner cảnh báo vàng cho câu trả lời về deadline/học bổng: "Thông tin này chưa được xác nhận chính thức. Vui lòng kiểm tra trang tuyển sinh."
- Nút "Nói chuyện với tư vấn viên" hiển thị cố định khi topic là deadline/học bổng/hồ sơ
- Chỉ số "Mức độ chắc chắn": Confirmed / Unconfirmed / Estimated
- Disclaimer cố định dưới chatbox: "Chatbot AI hỗ trợ thông tin tham khảo. Thông tin chính thức xem tại [link tuyển sinh]."

**Cách giải quyết root cause:** Khắc phục lớp UI — người dùng không còn mặc định tin AI như thông báo chính thức; có đường thoát rõ ràng sang kênh chính thức.

---

### L2 — Prompt (Chỉ dẫn AI)

**Vấn đề cần giải quyết:** Model tự sinh câu trả lời chắc chắn dù không có nguồn; không có cơ chế buộc từ chối hoặc escalate.

**Giải pháp:**
- System prompt yêu cầu: mọi thông tin về deadline/học bổng/học phí/điều kiện hồ sơ phải kèm theo `[Nguồn: ... | Ngày: ...]`
- Refusal template bắt buộc khi không có nguồn live: "Mình chưa có thông tin chính thức cập nhật về [chủ đề] cho [năm]. Bạn nên kiểm tra tại [link] hoặc liên hệ tư vấn viên."
- Escalation trigger: nếu topic là `deadline cụ thể năm hiện tại`, `học bổng có điều kiện cá nhân`, `hồ sơ thiếu/ngoại lệ` → bắt buộc offer counselor ngay trong response
- Câu từ chối áp lực: khi user nói "nói đại đi", "ước lượng thôi" → AI giải thích rõ rủi ro và từ chối đoán

**Cách giải quyết root cause:** Khắc phục lớp Model — guardrail trong prompt giảm xác suất hallucination chắc chắn; tạo hành vi từ chối nhất quán.

---

### L3 — Architecture (Kiến trúc dữ liệu)

**Vấn đề cần giải quyết:** Knowledge base chứa tài liệu cũ, hết hạn không được flagged; không có cơ chế tự động phát hiện stale data hoặc route escalation.

**Giải pháp:**
- Metadata tagging bắt buộc cho mỗi tài liệu: `expiry_date`, `source_url`, `document_type` (deadline/scholarship/tuition/general), `last_verified`
- Staleness detector: tài liệu về deadline/học bổng có `expiry_date` quá hạn hoặc `last_verified` > 30 ngày → bị flagged, không được RAG truy xuất; thay thế bằng fallback message
- Fallback handler: khi không tìm được tài liệu liên quan hoặc tài liệu bị flagged stale → trả về template từ chối chuẩn + link official + option gặp counselor
- Counselor routing API: khi AI trigger escalation → tự động tạo ticket/chat session với counselor, ghi log context của cuộc hội thoại
- Audit log: ghi lại mỗi câu hỏi về deadline/học bổng/hồ sơ và câu trả lời AI → reviewable bởi admissions team hàng tuần

**Cách giải quyết root cause:** Khắc phục lớp Input — đảm bảo model không bao giờ được truy xuất tài liệu lỗi thời; escalation được xử lý tự động thay vì phụ thuộc vào model.

---

## Sơ đồ ba lớp phối hợp

```
USER HỎI DEADLINE/HỌC BỔNG
         │
         ▼
┌─────────────────────────────────────────────────────────┐
│  L3 — Architecture                                      │
│  RAG truy xuất → kiểm tra metadata expiry_date         │
│  • Còn hạn + có source → trả về tài liệu + citation   │
│  • Hết hạn / không có → fallback handler               │
└──────────────────────┬──────────────────────────────────┘
                       │
         ┌─────────────┴──────────────┐
         │ Có tài liệu                │ Không có / stale
         ▼                            ▼
┌────────────────┐          ┌─────────────────────────────┐
│ L2 — Prompt    │          │ Fallback: từ chối chuẩn     │
│ Cite nguồn     │          │ + link official + counselor │
│ Escalation     │          └─────────────────────────────┘
│ trigger nếu    │
│ cần            │
└───────┬────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│  L1 — UI/UX                                             │
│  Hiển thị câu trả lời + badge nguồn + ngày cập nhật   │
│  Banner cảnh báo nếu topic là deadline/học bổng        │
│  Nút "Nói chuyện với tư vấn viên" luôn hiển thị        │
└─────────────────────────────────────────────────────────┘
```

---

## Vì sao cần cả ba lớp

| Nếu chỉ có... | Vẫn còn rủi ro |
|---|---|
| Chỉ L1 (UI) | User thấy cảnh báo nhưng AI vẫn có thể đưa số liệu bịa; cảnh báo bị bỏ qua |
| Chỉ L2 (Prompt) | Prompt guardrail có thể bị bypass; không giải quyết stale data ở tầng dữ liệu |
| Chỉ L3 (Architecture) | Stale data được kiểm soát nhưng model vẫn có thể hallucinate với dữ liệu có sẵn nếu không có prompt constraint |
| L1 + L2 | Vẫn có thể model nhận được tài liệu cũ và hallucinate một phần; không có fallback tự động |
| L1 + L3 | Model vẫn thiếu guardrail từ chối đoán hoặc escalate đúng cách |
| L2 + L3 | User không có tín hiệu UI rõ ràng về mức độ tin cậy; không có đường thoát sang counselor dễ thấy |
| **L1 + L2 + L3** | **Phòng thủ nhiều lớp: dữ liệu đúng → model hành xử đúng → user thấy tín hiệu rõ ràng** |
