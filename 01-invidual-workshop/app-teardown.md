# Workshop — Mổ App AI Thật

**Thời gian:** 35-45 phút  
**Hình thức:** cá nhân trước, chia sẻ theo nhóm sau  
**Output:** finding note + sketch `as-is / to-be`

Mục tiêu không phải chấm "UI đẹp hay xấu". Mục tiêu là dùng sản phẩm thật như một bài needfinding: tìm chỗ product gãy trong workflow thật, rồi viết finding đó thành quyết định product.

---

## 1. Chọn một sản phẩm để dùng thử

**Sản phẩm đã chọn: Vietnam Airlines — NEO**

| Sản phẩm | AI feature | Cách truy cập |
|---|---|---|
| MoMo — Moni | Trợ thủ tài chính, phân tích chi tiêu, chatbot | App MoMo |
| **Vietnam Airlines — NEO** ✅ | **Chatbot hỗ trợ vé, hành lý, khiếu nại** | **Website vietnamairlines.com / Zalo VNA** |
| V-App — V-AI | Trợ lý voice/text, gợi ý theo ngữ cảnh | App V-App |
| App theo track nhóm | App thật nhóm đang chọn cho hackathon | Cần screenshot/link |

---

## 2. Dùng thử: promise vs reality

### Product hứa gì?
NEO tự giới thiệu là "trợ lý ảo của Vietnam Airlines" với 4 tính năng chính được quảng bá:
- **Thông tin chuyến bay** — kiểm tra giờ cất/hạ cánh
- **Thông tin vé máy bay** — kiểm tra hành trình và tình trạng đặt chỗ
- **Tìm kiếm giá vé** — hỗ trợ tìm giá vé tốt nhất
- **Thông tin hành lý** — tra số kg ký gửi theo hành trình

Slogan ngầm: **hỏi NEO là xong, không cần vào web hay gọi hotline**.

### User nào được hứa sẽ được giúp?
Hành khách cần tra thông tin nhanh trước/trong chuyến đi — đặc biệt người không muốn chờ hotline hoặc người đang cân nhắc mua vé muốn so sánh giá.

### Kỳ vọng AI làm được task nào?
- Tìm và so sánh giá vé theo nhiều ngày để chọn ngày rẻ nhất.
- Tra trạng thái chuyến bay (on-time / delay / hủy).
- Kiểm tra tình trạng đặt chỗ theo mã booking.
- Giải đáp chính sách hành lý theo đúng loại vé của user.
- Hỗ trợ đặt vé từ đầu đến cuối trong chat.

### Khi dùng thật, điểm gãy xuất hiện ở đâu?

| # | Điểm gãy quan sát được (từ test thực tế) |
|---|---|
| 1 | NEO hỏi rating sự hài lòng **ngay lúc mở chat**, trước khi giúp được gì |
| 2 | Tìm giá 1 ngày cụ thể: hoạt động tốt. Nhưng hỏi **"ngày nào rẻ nhất trong tuần tới"** → NEO không hiểu range query, lại hỏi ngày cụ thể |
| 3 | Sau khi tìm được giá vé, user hỏi **"bạn có thể giúp tôi đặt vé không?"** → NEO từ chối, chuyển hướng ra website — toàn bộ context (chuyến bay, ngày, hành khách) **biến mất**, user phải nhập lại từ đầu |
| 4 | Disclaimer sau mỗi kết quả: *"Giá hiển thị chỉ mang tính chất tham khảo"* — user không biết có thể tin con số này không |
| 5 | Hỏi thông tin đặt chỗ: NEO yêu cầu mã booking + họ, nhưng **không thực sự tra được hệ thống** — chỉ hướng dẫn sang web |

### Evidence từ test thực tế

**Observation 1 — Rating prompt trước khi service (khi mở chat):**
> NEO vừa chào xong liền hỏi đánh giá 1–5 sao — conversation chưa có nội dung hỗ trợ thực tế.

**Observation 2 — Tìm giá 1 ngày: hoạt động đúng luồng ✅**
> Prompt: *"Vé máy bay Hà Nội đi Đà Nẵng cuối tuần này giá bao nhiêu?"*  
> NEO thu thập đủ thông tin theo từng bước (điểm đi → điểm đến → ngày → số khách → xác nhận), trả về 3 chuyến bay thực tế:
> - VN6071 (Pacific Airlines) 18:20 — 1,724,000 VND
> - VN7175 (Vietnam Airlines) 20:45 — 1,724,000 VND  
> - VN7191 (Vietnam Airlines) 21:30 — 1,724,000 VND

**Observation 3 — Range query: thất bại ❌**
> Prompt: *"Ngày nào trong tuần tới có giá rẻ nhất từ Hà Nội đi Đà Nẵng?"*  
> NEO xử lý như tìm vé 1 ngày thông thường, hỏi lại *"Ngày đi (Trong tuần tới)"* — không hiểu đây là câu hỏi so sánh nhiều ngày. Không có calendar view, không có multi-date search.

**Observation 4 — Booking handoff: drop context ❌**
> Sau khi NEO đã tìm được 3 chuyến bay, user hỏi: *"Bạn có thể giúp tôi đặt vé không?"*  
> NEO trả lời: hướng dẫn vào website hoặc app, liệt kê lợi ích đặt online — **không đặt được trong chat**, không truyền context sang web, user phải điền lại từ đầu.

**Observation 5 — Hành lý theo loại vé: phân biệt được ✅**
> Prompt: *"Tôi bay hạng Eco Saver, hành lý ký gửi có bao gồm trong giá vé không?"*  
> NEO phân biệt được Eco Saver vs Economy thông thường cho nội địa. Với quốc tế thì vẫn hedge: *"vui lòng kiểm tra kỹ điều kiện vé"*.

---

## 3. Vẽ 4 paths

| Path | Quan sát trong NEO |
|---|---|
| **Happy** | User hỏi *"hành lý Economy ký gửi bao nhiêu kg?"* → NEO trả đúng, chi tiết theo từng hành trình ✅. Tìm vé 1 ngày cụ thể với đủ thông tin → trả kết quả thực tế ✅ |
| **Low-confidence** | Trong luồng tìm vé có cấu trúc: NEO biết hỏi lại từng trường còn thiếu (ngày đi, số khách) ✅. Tuy nhiên với **range query mơ hồ** ("ngày nào rẻ nhất trong tuần tới"): NEO không nhận ra đây là câu hỏi so sánh, vẫn yêu cầu 1 ngày cụ thể ❌ |
| **Failure** | Sau khi tìm giá xong, user muốn đặt vé → NEO không thực hiện được, handoff ra website mà **không truyền context** (chuyến bay, ngày, hành khách) → user phải làm lại từ đầu ❌ |
| **Correction** | Không test được trực tiếp trong session này. Nhưng disclaimer *"Giá hiển thị chỉ mang tính chất tham khảo"* cho thấy NEO tự nhận không đảm bảo độ chính xác mà **không có cơ chế nào để user biết khi nào nên tin** ❌ |

**Tóm tắt:** Happy path hoạt động tốt trong luồng có cấu trúc (1 ngày, đủ params). Vỡ khi query vượt ra ngoài khuôn (range, booking thật, đặt vé end-to-end).

---

## 4. Viết finding thành quyết định

### Finding 1 — Range query bị xử lý như single-date query

```
Khi user hỏi "Ngày nào trong tuần tới có giá rẻ nhất từ Hà Nội đi Đà Nẵng?",
AI nhận diện intent là tìm vé thông thường thay vì hiểu đây là câu hỏi so sánh nhiều ngày,
hậu quả là user không có được thông tin để chọn ngày bay tiết kiệm — mục đích gốc không được đáp ứng.
Lỗi thuộc layer Intent (không phân biệt single-date search vs range/comparison query).
Nên sửa bằng: nhận diện pattern "ngày nào rẻ nhất / linh hoạt ngày" → chuyển sang
calendar price view (hiển thị giá theo tuần) thay vì hỏi ngày cụ thể.
```

### Finding 2 — Booking handoff mất toàn bộ context

```
Khi user đã hoàn thành tìm kiếm trong chat (có chuyến bay, ngày, số hành khách)
và yêu cầu "đặt vé ngay",
AI chuyển hướng ra website/app mà không truyền context,
hậu quả là user phải điền lại toàn bộ thông tin từ đầu — friction tăng, tỷ lệ abandon tăng.
Lỗi thuộc layer UX Recovery + Promise (hứa hỗ trợ đặt vé nhưng thực ra chỉ redirect).
Nên sửa bằng: deep-link kèm pre-filled params (origin, destination, date, pax)
hoặc handoff token để website/app tự điền lại.
```

### Finding 3 — Disclaimer "chỉ mang tính tham khảo" không có ngưỡng rõ ràng

```
Khi NEO trả về kết quả giá vé kèm disclaimer "chỉ mang tính tham khảo",
user không biết độ tin cậy cụ thể là bao nhiêu (data cũ bao lâu? sai bao nhiêu %?),
hậu quả là user không có cơ sở để quyết định có cần verify lại không — có thể bỏ lỡ deal hoặc bị bất ngờ khi giá thay đổi.
Lỗi thuộc layer Safety + Trust calibration.
Nên sửa bằng: thêm timestamp cụ thể ("cập nhật lúc 14:32") và ghi rõ
"giá có thể thay đổi trong vòng X phút" thay vì disclaimer chung chung.
```

### Finding 4 — Rating prompt sai thời điểm

```
Khi user vừa mở chat và chưa nhận được bất kỳ hỗ trợ nào,
NEO hỏi satisfaction rating ngay trong lần đầu tương tác,
hậu quả là UX bị gián đoạn trước khi bắt đầu — user chưa có trải nghiệm gì để đánh giá.
Lỗi thuộc layer UX / Promise.
Nên sửa bằng: chỉ trigger rating prompt sau khi conversation kết thúc
(user không gửi thêm tin nhắn trong 5 phút, hoặc bấm "Đã giải quyết").
```

---

## 5. Sketch as-is / to-be

### As-is (flow hiện tại — dựa trên test thực tế)

```
User mở chat
       |
       v
NEO chào + HỎI RATING NGAY  ← [ĐIỂM GÃY 4: sai thời điểm]
       |
       v
User gõ câu hỏi
       |
       +-- Tìm vé 1 ngày (đủ params) ──────────────────────────────────────── ✅ HAPPY PATH
       |       |
       |       └── Trả 3 chuyến + giá + disclaimer "chỉ tham khảo" ← [GAP 3: trust mờ]
       |               |
       |               └── User: "Đặt vé giúp tôi" ──────────────────────── ❌ FAILURE PATH
       |                           |
       |                           └── NEO: redirect web, không truyền context ← [ĐIỂM GÃY 2]
       |
       +-- Range query ("ngày nào rẻ nhất?") ─────────────────────────────── ❌
       |       |
       |       └── NEO hỏi ngày cụ thể, không hiểu range intent ← [ĐIỂM GÃY 1]
       |
       +-- Hỏi hành lý (câu rõ ràng) ─────────────────────────────────────── ✅ HAPPY PATH
       |
       +-- Kiểm tra đặt chỗ ──────────────────────────────────────────────── ⚠️ PARTIAL
               |
               └── NEO hỏi booking code + họ (đúng) nhưng không tra được hệ thống thật
```

### To-be (flow đề xuất)

```
User mở chat
       |
       v
NEO chào — phân loại intent nhanh:
"Bạn cần: [Tìm vé] [Tra chuyến bay] [Hành lý] [Đặt chỗ] [Khác]?"
       |
       +-- Tìm vé ──────────────────────────────────────────────────────────────────────
       |       |
       |       +-- Query 1 ngày: thu thập params → search → trả kết quả + timestamp giá ✅
       |       |
       |       +-- Range query ("linh hoạt / rẻ nhất") → Calendar price view (7 ngày) ✅
       |               |
       |               └── User chọn ngày → confirm → deep-link đặt vé (pre-filled) ✅
       |                           [KHÔNG bắt user điền lại]
       |
       +-- Hành lý ─── Phân biệt loại vé + hành trình → trả đúng policy ✅
       |
       +-- Kiểm tra đặt chỗ ─── Hỏi booking code → tra hệ thống thật → trả status ✅
       |
       └── [Sau khi user không gửi thêm tin nhắn 5 phút] → HỎI RATING ✅
```

---

## 6. Tự kiểm trước khi nộp

- [x] Có ít nhất 1 screenshot hoặc observation cụ thể.  
  → 5 observations từ test thực tế: rating prompt, kết quả tìm vé 3 chuyến (VN6071/7175/7191 — 1,724,000 VND), range query failure, booking handoff drop, hành lý Eco Saver.

- [x] Có đủ 4 paths hoặc nói rõ path nào chưa có trong product.  
  → Happy path hoạt động trong luồng có cấu trúc. Low-confidence thiếu với range query. Failure path là booking handoff mất context. Correction path chưa test được — được ghi rõ.

- [x] Finding được viết thành product decision, không chỉ là nhận xét.  
  → 4 findings đều theo format: trigger → failure → impact → layer → fix.

- [x] Sketch có as-is và to-be.  
  → Đã vẽ flowchart cho cả hai, phản ánh đúng test thực tế.

- [x] Có một câu nói rõ finding này sẽ đổi gì trong SPEC.  
  → **Finding quan trọng nhất cho SPEC (Finding 2):** NEO drop toàn bộ context khi handoff sang website sau khi đã tìm được vé. Nếu build prototype, điểm cần sửa ngay là deep-link pre-filled hoặc handoff token — giúp user không phải điền lại thông tin và giảm abandon rate tại điểm chuyển đổi quan trọng nhất trong funnel đặt vé.
