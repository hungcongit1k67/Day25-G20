# 00 — Bối cảnh sản phẩm

**Dùng file này làm ngữ cảnh đầu tiên cho mọi prompt AI trong Day 25.**

---

## Sản phẩm

**Chatbot AI tư vấn tuyển sinh** đặt trên website tuyển sinh chính thức của một trường đại học tư thục tại Việt Nam, phục vụ mùa tuyển sinh 2025–2026.

---

## Workflow

| Trường | Mô tả |
|---|---|
| **AI làm gì** | Trả lời câu hỏi về ngành học, học phí, học bổng, deadline nộp hồ sơ, điều kiện xét tuyển, giấy tờ cần chuẩn bị và hướng dẫn đăng ký tư vấn. AI là điểm hỗ trợ đầu tiên, dẫn user đến link/kênh liên hệ phù hợp. |
| **AI KHÔNG làm** | Xác nhận trúng tuyển, cam kết cấp học bổng, tự tạo deadline, thay counselor đưa lời hứa chính thức, nhận/nộp hồ sơ thay học sinh, yêu cầu thông tin nhạy cảm không cần thiết. |

---

## Người dùng

- Học sinh lớp 12, thí sinh tự do đang tìm hiểu đại học
- Phụ huynh/người giám hộ cân nhắc học phí và học bổng
- Giai đoạn: 1–3 tháng trước deadline tuyển sinh/học bổng, hoặc sát hạn nộp hồ sơ

---

## Kênh & Ngữ cảnh

Chatbot hiển thị trên website tuyển sinh chính thức → user dễ tin câu trả lời AI tương đương thông báo chính thức từ trường.

---

## Primary failure (từ Day 24)

> **Hallucination về deadline, học bổng, điều kiện hồ sơ và học phí.**
>
> Khi user hỏi thông tin tuyển sinh có tính quyết định (deadline, học bổng, điều kiện hồ sơ, học phí) trong một năm cụ thể, AI có xu hướng bịa câu trả lời chắc chắn thay vì thừa nhận thiếu nguồn chính thức và chuyển sang kênh xác nhận — gây rủi ro lỡ deadline, mất cơ hội học bổng và ra quyết định tài chính sai cho học sinh/phụ huynh.

---

## 3 Failure candidates (từ Day 24)

| ID | Tên | Layer chính | Severity |
|---|---|---|---|
| C1 | Hallucination về deadline/học bổng/hồ sơ | Input | High |
| C2 | Sycophancy — chiều lòng user dù thông tin sai | Model | Medium–High |
| C3 | Escalation failure — không chuyển case nhạy cảm sang người thật | Human review | High |

---

## Safety Question

> Trong chatbot tư vấn tuyển sinh đại học dùng bởi học sinh lớp 12 và phụ huynh trên website tuyển sinh chính thức trước mùa nộp hồ sơ/học bổng, AI có bịa hoặc xác nhận chắc chắn deadline, học bổng, điều kiện hồ sơ hoặc học phí khi user hỏi thông tin của một năm tuyển sinh cụ thể nhưng hệ thống không có nguồn chính thức/cập nhật không, gây hậu quả cho học sinh là lỡ hạn nộp, mất cơ hội học bổng, chuẩn bị sai hồ sơ và khiến phụ huynh ra quyết định tài chính sai?
