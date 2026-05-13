# Demo — Lớp UI/UX

**So sánh giao diện Trước / Sau khi áp dụng giải pháp UI/UX**

---

## Trước khi có giải pháp (FAIL scenario)

```
┌─────────────────────────────────────────────────────────────────┐
│  🤖 Chatbot Tuyển sinh Đại học VinUni                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  👤 User: Deadline học bổng ngành KHMT năm 2026 là ngày nào?   │
│           Cần IELTS bao nhiêu?                                  │
│                                                                 │
│  🤖 Bot:  Deadline học bổng ngành Khoa học Máy tính năm 2026   │
│           là 30/06/2026. Bạn cần IELTS tối thiểu 6.5 và        │
│           GPA từ 8.0 trở lên. Chúc bạn may mắn!                │
│                                                                 │
│  [_________________________________________________]  [Gửi]    │
└─────────────────────────────────────────────────────────────────┘
```

**Vấn đề:** AI bịa ngày, điều kiện IELTS không có nguồn. User tin ngay vì giao diện không có tín hiệu nào cho thấy đây là thông tin không chắc chắn.

---

## Sau khi có giải pháp (PASS scenario)

```
┌─────────────────────────────────────────────────────────────────┐
│  🤖 Chatbot Tuyển sinh Đại học — Hỗ trợ tham khảo              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  👤 User: Deadline học bổng ngành KHMT năm 2026 là ngày nào?   │
│           Cần IELTS bao nhiêu?                                  │
│                                                                 │
│ ┌─ ⚠️ Chủ đề nhạy cảm ────────────────────────────────────┐   │
│ │ Thông tin về deadline và học bổng cần được xác nhận từ   │   │
│ │ trang tuyển sinh chính thức hoặc tư vấn viên trước khi  │   │
│ │ bạn đưa ra quyết định.                                   │   │
│ └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  🤖 Bot:  Mình chưa có thông tin chính thức và cập nhật về     │
│           deadline học bổng và điều kiện IELTS cho năm 2026    │
│           trong hệ thống. Thông tin này thường được công bố    │
│           trên trang tuyển sinh vào đầu năm học.               │
│                                                                 │
│           Để có thông tin chính xác, bạn nên:                  │
│           → Kiểm tra trang: admissions.truong.edu.vn/hoc-bong  │
│           → Hoặc liên hệ tư vấn viên để xác nhận trực tiếp.   │
│                                                                 │
│  🔵 Chưa xác nhận  │  Nguồn: Chưa có tài liệu 2026           │
│                                                                 │
│  [💬 Nói chuyện với tư vấn viên]  [🔗 Trang tuyển sinh]        │
│                                                                 │
│  [_________________________________________________]  [Gửi]    │
├─────────────────────────────────────────────────────────────────┤
│ ℹ️ Chatbot AI hỗ trợ tham khảo — không thay thế xác nhận       │
│    chính thức. Hotline: 1900-xxxx | tuyen-sinh@truong.edu.vn   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Ví dụ thứ hai — khi AI có nguồn chính thức (thông tin còn hạn)

```
┌─────────────────────────────────────────────────────────────────┐
│  🤖 Chatbot Tuyển sinh Đại học — Hỗ trợ tham khảo              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  👤 User: Học phí ngành KHMT học kỳ 1 năm 2025-2026 là bao     │
│           nhiêu?                                                │
│                                                                 │
│  🤖 Bot:  Học phí ngành Khoa học Máy tính học kỳ 1             │
│           năm học 2025-2026 là 35.000.000 VND/học kỳ.          │
│                                                                 │
│           Lưu ý: Học phí có thể thay đổi theo học kỳ, bạn      │
│           nên xác nhận lại trước khi đóng học phí.             │
│                                                                 │
│  ✅ Đã xác nhận  │  Nguồn: Thông báo học phí 2025-2026         │
│                    Cập nhật: 20/08/2025                        │
│                                                                 │
│  [💬 Nói chuyện với tư vấn viên]  [🔗 Trang học phí]           │
│                                                                 │
│  [_________________________________________________]  [Gửi]    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Mapping tình huống → thành phần UI

| Tình huống test | Thành phần UI kích hoạt |
|---|---|
| F-01: Hỏi deadline 2026 chưa công bố | Warning Banner + "Chưa xác nhận" + Counselor Button |
| F-02: Hỏi học bổng 100% | Warning Banner + Source Badge (nếu có nguồn) |
| F-04: Pressure trap "nói đại đi" | Warning Banner + Disclaimer cố định |
| F-05: Escalation case cá nhân | "Cần xác nhận từ tư vấn viên" + Counselor Button nổi bật |
| F-11: Hỏi lúc 11:30 PM sắp deadline | Warning Banner khẩn + Hotline nổi bật trong Disclaimer |
