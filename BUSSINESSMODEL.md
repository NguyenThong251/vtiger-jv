# FCJ Business Model — Lead → Deal → Quote (Vtiger 8.4)

Tài liệu mô tả nghiệp vụ và workflow; phần **kỹ thuật triển khai, field types, plan stack** nằm trong [PLAN_STACK_FCJ_VTIGER.md](./PLAN_STACK_FCJ_VTIGER.md). Bản đồ developer và link tài liệu chính thức: [vtlib/Vtiger/NOTE.md](./vtlib/Vtiger/NOTE.md).

## Ánh xạ ngôn ngữ nghiệp vụ → core Vtiger

| Thuật ngữ trong tài liệu | Module Vtiger | Bảng / custom |
|--------------------------|---------------|----------------|
| Deal | **Potentials** | `vtiger_potential`, **`vtiger_potentialscf`** (field custom) |
| Quote | **Quotes** | `vtiger_quotes` (+ scf nếu có) |
| Lead | **Leads** | `vtiger_leaddetails`, `vtiger_leadscf` |
| Organization | **Accounts** | `vtiger_account` |
| Contact | **Contacts** | `vtiger_contactdetails` |

**Lưu ý:** Đổi nhãn “Opportunity” thành “Deal” bằng **ngôn ngữ / label**, không cần đổi tên module nếu muốn giữ đường upgrade chuẩn. Các cột boolean/remind phải tạo qua **Settings → Module Layout & Fields** hoặc script **vtlib** (metadata + cột trên `vtiger_potentialscf`). Không dùng tên bảng `vtiger_dealscf` — **không tồn tại** trong core Potentials.

## Tiêu chí kỹ thuật khi làm tính năng (tóm tắt)

1. **Clean code** — tách custom ra extension / `custom/`, không sửa lõi lan man.  
2. **Workflow + DB sẵn có** — ưu tiên WF có sẵn + cron; chỉ custom task khi cần.  
3. **Scale** — điều kiện WF rõ, cờ chống trùng, scheduled chia module.  
4. **Security / chuẩn** — tôn trọng profile, không bypass quyền khi tự động hóa.  
5. **Upgrade** — hạn chế sửa file trong `modules/` gốc; can thiệp core chỉ khi bắt buộc và ghi chú merge.

---

<!-- GIẢI THÍCH -->

Bắt đầu Deal (Opportunity) và Quote
Opportunity Name: Tên cơ hội bán hàng "Opportunity Name
change lại thành Deal Name" Yes Deal Details Deal "sử dụng của vtiger
Nên sữa Opportunity thành Deal"
Organization Name: Công ty khách hàng Organization Name "Reference / Lookup field
đến Organization module" No Deal Details Deal sử dụng của vtiger
Contact Name: Người liên hệ phía khách hàng Contact Name "Reference / Lookup field
đến Contact module" No Deal Details Deal sử dụng của vtiger
Amount: Giá trị Deal dự kiến Amount No Deal Details Deal sử dụng của vtiger
結果 (Cột S): Status. ví dụ thất bại với KH Deal Stage (Sales Stage) Picklist "Prospecting
Needs Analysis
Proposal / Price Quote
Follow-up 1
JP Calling
Follow-up 2
Final Follow-up
Negotiation or Review
Closed Won
Closed Lost" Yes Deal Details Deal sử dụng của vtiger
Note: Chú thích lý do status của Deal Note Text Area No Deal Details Deal
Expected Close Date: Ngày dự kiến chốt Deal Expected Close Date Yes Deal Details Deal sử dụng của vtiger
Assigned To: Người phụ trách Deal Assigned To Reference to User/Group Yes Deal Details Deal sử dụng của vtiger
Probability: Xác suất chốt Deal Probability No Deal Details Deal sử dụng của vtiger
Lead Source: Nguồn lead đến từ đâu Lead Source "Reference / Lookup field
đến Lead module" No Deal Details Deal sử dụng của vtiger
"Campaign Source: Chiến dịch marketing liên quan

- Chỉ cần nếu bạn có dùng Campaign trong CRM" Campaign Source "Reference / Lookup field
  đến Campaign module" No Deal Details Deal sử dụng của vtiger
  (2)見積もり送付 (Cột O): Ngày gửi email báo giá Quote Sent Date Date No Deal Details Deal
  "(2)-1リマインド (Cột P):
  Sau 2 ngày kể từ ngày gửi mail báo giá. Gửi tiếp mail remind lần 1" First Reminder Date Date No Deal Details Deal
  "(3)田村さんにチャット (Cột Q):
  Ngày nhắn cho ông bên Nhật để ổng call khách hàng (rule sau 1 ngày remind lần 1)" JP Call Request Date Date No Deal Details Deal
  "(3)田村さんから電話(Cột R):
  Ngày mà ông Nhật gọi điện cho K/H (ko biết chính xác ngày ổng gọi)
  nên khi ổng call thì ổng tự điền cột này.
  Nếu ổng không call dc thì ổng không điền, tức là mất liên lạc với K/H" JP Call Date Date No Deal Details Deal
  備考(失注理由など) (Cột T): Chú thích thất bại. K/H nhờ cty khác làm rồi Failure Reason Text Area No Deal Details Deal
  "(1)からの経過日数 (Cột U):
  Số ngày đã trôi qua kể từ (1) 一次対応 (Cột M) / ngày FCJ gửi mail trả lời đầu tiên" Days Since First Response Integer 10 No Deal Details Deal
  "(2)からの経過日数 (Cột V):
  Số ngày đã trôi qua kể từ (2) 見積もり送付 (Cột M) / ngày gửi email báo giá" Days Since Quotation Sent Integer 10 No Deal Details Deal
  "4.リマインド (Cột W):
  Remind lần 2 (rule remind lần 2 sau 2 tuần kể từ ngày nhắn cho ông Nhật gọi điện)" Second Reminder Date Date No Deal Details Deal
  "5.キャンペーン (Cột X):
  Thêm 2 tuần nữa, tức là 1 tháng kể từ ngày nhắn ông Nhật gọi điện
  thì TrangLK bắt đầu gửi khách campaign FREE" Third Reminder Date Date No Deal Details Deal
  "結果 (Cột Y): Kết quả chiến dịch
- メール不通
- 受注 2024/09/17
- 返信無し" Campaign Result Text 255 No Deal Details Deal
  ヒアリング (Cột Z): Ngày nhận đơn hàng Order Received Date Date No Deal Details Deal
  Remind1 Created: false (default) "Tạo field Checkbox trên Potentials → cột trên vtiger_potentialscf (vd: remind1_created); ưu tiên Module Manager / vtlib, tránh ALTER tay ngoài migration." No Deal Details Deal
  Remind2 Created: false (default) "Giống trên — vtiger_potentialscf.remind2_created" No Deal Details Deal
  Remind3 Created: false (default) "Giống trên — vtiger_potentialscf.remind3_created" No Deal Details Deal
  JP Call Requested: false (default) "Giống trên — vtiger_potentialscf.jp_call_requested" No Deal Details Deal
  "Weighted Revenue: Doanh thu có trọng số
  Thường = Amount x Probability
  Field này thiên về báo cáo hơn là nhập liệu hằng ngày, nên thường có thể ẩn." Weighted Revenue No Deal Details Deal "sử dụng của vtiger
  (tạm thời ẩn)"
  Probability: Xác suất chốt Deal Probability No Deal Details Deal "sử dụng của vtiger
  (tạm thời ẩn)"
  Description: Ghi chú mô tả Deal Description No Description Details Deal sử dụng của vtiger

## Workflow — hướng dẫn triển khai kỹ thuật

Phần dưới là **kịch bản nghiệp vụ** (Phase 1–4). Khi cấu hình trên CRM, đối chiếu:

- **Bảng vấn đề / giải pháp** (cron scheduled, cờ chống trùng, dừng remind khi Closed, Quote vs Deal): [PLAN_STACK_FCJ_VTIGER.md §6](./PLAN_STACK_FCJ_VTIGER.md#6-workflow--tổng-hợp-vấn-đề--cách-xử-lý--hướng-làm).  
- **Thứ tự task** T0–T8: [PLAN_STACK_FCJ_VTIGER.md §7](./PLAN_STACK_FCJ_VTIGER.md#7-plan-stack--thứ-tự-task-để-bạn-giao-việc-từng-phần).  
- **Case thiếu module/file**: workflow không được tham chiếu field/module chưa tồn tại; bật module phụ (SMS, Documents, …) trước khi gắn task tương ứng.

<!-- WORKFLOW -->

🔵 PHASE 1 — INQUIRY (LEAD STATUS)
KỊCH BẢN 0: Tạo Lead WORKFLOW 0.1 — Nhắc nếu Not Contacted quá 24h.
Bối cảnh Type: Scheduled Workflow
Lead mới tạo từ Cold Call / Webform Run: Daily
Condition:
1️⃣ Lead _ + Lead Status = Not Contacted
Nhập đầy đủ: Name, Phone, Email, Source + Customer Date IS NOT EMPTY
Gán Assigned To: Sales + Today ≥ Customer Date + 1 day
Lead Status: Not Contacted + Lead Reminder Sent = false (tránh tạo task trùng)
Update Response Status: Mới tạo, chưa xử lý (chưa Call) Action (auto):
Customer Date: Ngày khách gửi yêu cầu đầu tiên 1. Create Task: + Subject: “Follow-up Lead – Not Contacted > 24h”
2️⃣ Task _ + Assigned to: Lead Owner (Sales)
Tạo Task: Call (Email) trong vòng 24h + Task Type: Call
Task Status: Pending + Priority: High
Task Type: Call 2. Update Lead: + Lead Reminder Sent = true 3. Send Email to Lead Owner: [FCJ] Lead chưa xử lý sau 24h
KỊCH BẢN 1: Sales (W.D) có call K/H nhưng chưa liên hệ được _\*\*
Bối cảnh WORKFLOW 1.1 — Nhắc K/H nếu status: Attempted to Contact quá 48h.
Sales gọi K/H chưa bắt máy Type: Scheduled Workflow
Run: Daily
1️⃣ Lead _ Condition:
Lead Status: Attempted to Contact + Lead Status = Attempted to Contact
Update Response Status: “Gọi ngày 6/2 – K/H chưa bắt máy” + Today ≥ Modified Time + 2 days
Action (auto):
2️⃣ Task _ 1. Create Task:
Tạo Task: Call lại K/H trong vòng 2 days + Subject: “Follow-up Call again – Attempted to Contact > 48h”
Task Status: Open + Assigned to: Lead Owner (Sales)
Task Type: Call + Task Type: Call
Due date: Sau 2 ngày khi call lần đầu không bắt máy + Priority: High 2. Send Email to Lead Owner: [FCJ] Lead Call lại K/H sau 48h
3️⃣ Các module khác
Event ❌ (chưa có hẹn)
Deal ❌
Quote ❌
KỊCH BẢN 2: Sales (W.D) gọi được K/H – khách hỏi sơ bộ, chưa hẹn meeting
Bối cảnh
Sales gọi được K/H
Khách hỏi sơ qua dịch vụ nhưng chưa sẵn sàng có cuộc họp
1️⃣ Lead _
Lead Status: Contacted
Update Response Status: “Khách hỏi dịch vụ, cần trả lời thêm từ K/H”
Update field (FCJ Response Date): "Ngày bên FCJ gửi mail trả lời đầu tiên cho khách"
2️⃣ Task
Tạo Task: Follow-up waiting reply Customer
Task Status: Open
Task Type: Follow-up
3️⃣ Documents (optional)
Upload: Company Profile.pdf
Link document với Lead
👉 Vì: sau này Manager vào biết đã gửi gì
4️⃣ SMS Message (optional)
SMS: "Em (Sales) vừa gửi profile qua email, anh check giúp em nhé"
👉 Vì: tăng tỉ lệ khách mở mail
❌ Deal / Event
Chưa tạo
🟡 PHASE 2 — MEETING & QUALIFICATION
KỊCH BẢN 3: Chat trực tiếp với K/H để setup buổi meeting (sau khi call được K/H hỏi sơ bộ)
Bối cảnh
Khách chat trực tiếp
Quan tâm rõ dịch vụ
1️⃣ Live Chat (optional)
Live Chat Status: Completed
Nội dung: “Cần meeting + báo giá”
2️⃣ Lead _
Lead Status: In Progress
Update Response Status: “Khách chủ động chat, nêu rõ nhu cầu”
Update MTG: Chọn 有 (Yes)
Update Meeting Date: 03-20-2026 (Ngày Khách yêu cầu meeting)
3️⃣ Task _
Task: Setup meeting online
Status: Open
4️⃣ Event (Meeting) \*
Event: Meeting online
Event Status: Planned (đã lên kế hoạch)
Thời gian: 8/2 10:00
→ 3.1: W.D (TrangLK) gửi email confirm nếu chưa clear (nếu đã rõ yêu cầu từ k/h thì bỏ qua bước 3.1)
Bối cảnh
Khách trả lời nhưng:

- Nội dung chưa rõ
- Thiếu file
- Chưa rõ yêu cầu

1️⃣ Lead _
Lead Status: In Progress
Update field (FCV Response Date): "Ngày W.D (TrangLK) gửi email confirm cho khách nếu chưa clear"
Update Response Status: "Yêu cầu chưa rõ"
2️⃣ Task _
Task: W.D (TrangLK) gửi mail confirm
Status: Open
3️⃣ SMS Message
SMS: "Em (W.D) vừa gửi confirm yêu cầu qua email, anh check giúp em nhé"
👉 Mục đích: tăng tỉ lệ khách mở mail
KỊCH BẢN 4: Kết quả sau khi meeting
→ 4.1: Không tiềm năng
Bối cảnh
Đã meeting xong
Khách hàng không làm hoặc báo làm với công ty khác, hoặc không đủ budget
1️⃣ Event
Event Status: Held (đã được tổ chức)
2️⃣ Lead _
Lead Status: Lost Lead (Kết thúc mà không convert Lead)
Update Response Status: “lý do thất bại là K/H nhờ cty khác làm rồi”
→ 4.2: Xác định K/H tiềm năng → Convert Lead WORKFLOW 4.3 — Change Lead Status: Qualified (ưu tiên sau) WORKFLOW 4.4 — Thực hiện manual convert Lead ( auto check Deal )
Bối cảnh Trigger: Lead Status = Qualified and submit btn Save Trigger: Thực hiện convert manual Lead (click btn Convert Lead)
Đã meeting xong Action (auto convert Lead): Condition: Lead Status != Qualified
Khách hàng có budget 1. Create Contact (popup confirm) Action (auto): 2. Create Organization (popup confirm) 1. Lead Status = Qualified
1️⃣ Event 3. Create Deal (popup confirm) 2. Notifi slack channel
Event Status: Held (đã được tổ chức) 4. Notifi slack channel 3. Notify to hệ thống 5. Notify to hệ thống
2️⃣ Task _
Task: Update Meeting Outcome
Status: Open
3️⃣ Lead _ (Cần báo giá chính thức)
Lead Status: Qualified
Update Response Status: “K/H có budget, cần proposal”
4️⃣ Convert Lead _
Sau khi convert:
✅ Contact
✅ Organization
✅ Deal
🔴 PHASE 3 — QUOTE & FOLLOW-UP (DEAL STAGE)
Từ đây trở đi KHÔNG nằm ở Lead nữa.
KỊCH BẢN 5: Deal được tạo – chuẩn bị báo giá
Bối cảnh
Đã convert
Có Deal bán hàng
Đang phân tích yêu cầu chi tiết.
FCJ mapping
Estimator đang estimate để đưa ra công số
Đang hỏi nội bộ kỹ thuật
1️⃣ Deal _
Deal Stage: Needs Analysis
Amount: 500,000,000 VND
2️⃣ Task _
Tạo Task: Chuẩn bị giải pháp & proposal để soạn proposal (đề xuất)
Task Status: Completed
3️⃣ Documents (optional)
Upload: Proposal_v1.pdf
Link document với Deal
KỊCH BẢN 6: Tạo Quote (báo giá) cho Deal và gửi cho K/H
Bối cảnh 6.1 WORKFLOW 1 — Khi gửi email báo giá (Cột O) 6.3 WORKFLOW 3 — (Cột Q) Request JP Call = (Cột P) Remind #1 + 1 ngày 6.5 WORKFLOW 5 — (Cột X) Remind #3 / Campaign FREE = (Cột Q) Request JP Call + 28 ngày
Khách yêu cầu gửi báo giá. Trigger: When Quote is updated CÁCH 1 — Theo ngày (không phụ thuộc task completion) Loại: Scheduled Workflow
Đã gửi báo giá chính thức. Condition: Quote Stage = Delivered Loại: Scheduled Workflow Condition:
Action (auto): Condition: + JP Call Requested = true
1️⃣ Deal _ 1. Update field Quote Sent Date = now() + Remind1 Created = true + Remind3 Created = false
Deal Stage: Proposal / Price Quote 2. Update field Deal Stage: Proposal / Price Quote + JP Call Requested = false + Today ≥ JP Call Request Date + 28 days 3. Reset: + Today ≥ First Reminder Date + 1 day + Deal Stage NOT IN (Negotiation or Review, Closed Won, Closed Lost) → Nếu JP gọi và deal chuyển sang Negotiation, thì Workflow Remind #3 không nên chạy.
2️⃣ Quote _ Remind1 Created = false + Deal Stage NOT IN (Closed Won, Closed Lost) → vẫn chưa đóng Action (auto):
Quote Stage: Delivered Remind2 Created = false Action (auto): 1. Create Task:
Quote Sent Date: "Ngày gửi email báo giá" Remind3 Created = false 1. Create Task: + Subject: “Final Follow-up / Free Campaign”
Gắn với Deal JP Call Requested = false + Subject: “Request JP Call” + Assigned to: Sales
Giá: 500M + Assigned to: JP Owner 2. Update Deal:
6.2 WORKFLOW 2 — (cột P) Remind #1 = (Cột O) Ngày gửi mail báo giá + 2 ngày 2. Update Deal: + Third Reminder Date = Today (now())
3️⃣ Email / SMS Loại: Scheduled Workflow + JP Call Request Date = Today (now()) + Remind3 Created = true
Gửi Email Quote Run: Daily + JP Call Requested = true 3. Update Deal Stage → “Final Follow-up”
Gửi SMS: "Em đã gửi báo giá, anh check giúp em" Condition: 3. Update Deal Stage → “JP Calling” 4. Send Email Notification to Sales (W.D) + Quote Stage = Delivered 4. Send Notification to JP 5. Add to Marketing List: “Free Campaign” hoặc tạo record Campaign/Email sequence
"cần workflow khi JP Called thì updated JP Call Date: now()
và change Deal Stage: Negotiation or Review" + Remind1 Created = false + Today ≥ Quote Sent Date + 2 days 6.4 WORKFLOW 4 — (Cột W) Remind #2 = (Cột Q) Request JP Call + 14 ngày + Deal Stage NOT IN (Closed Won, Closed Lost) → vẫn chưa đóng Loại: Scheduled Workflow
Action (auto): Condition: 1. Create Task: + JP Call Requested = true + Subject: “Remind #1 – Follow-up Quote” + Remind2 Created = false + Assigned to: Deal Owner + Today ≥ JP Call Request Date + 14 days + Due date = Today + Deal Stage NOT IN (Negotiation or Review, Closed Won, Closed Lost) → Nếu JP gọi và deal chuyển sang Negotiation, thì Workflow Remind #2 không nên chạy. 2. Update Deal: Action (auto): + First Reminder Date = Today (now()) 1. Create Task: + Remind1 Created = true + Subject: “Remind #2 – Follow up” 3. Update Deal Stage → “Follow-up 1” + Assigned to: Sales 4. Send Email Notification to Sales (W.D) 2. Update Deal: + Second Reminder Date = Today (now()) + Remind2 Created = true 3. Update Deal Stage → “Follow-up 2” 4. Send Email Notification to Sales (W.D)
KỊCH BẢN 7: Negotiation or Review (đàm phán / review) lại giá với K/H
Bối cảnh
Khách phản hồi báo giá: (mục đích điều chỉnh báo giá)

- xin giảm giá
- đổi scope
- kéo timeline
- cần approval nội bộ

1️⃣ Deal _
Deal Stage: Negotiation or Review
Update (Note): điều khoản đang đàm phán
(Optional) Approval nếu giảm giá > X%
2️⃣ Quote _
Quote Stage có thể:

- Delivered (vẫn Delivered, đang thương lượng)
- hoặc tạo Quote Version 2 (khuyên dùng khi thay đổi giá/scope)

3️⃣ Tasks/Events

- Task: “Negotiate v2” (Open)
- Event: “Negotiation meeting” (Planned/Held)

✅ Gợi ý rule:

- Mỗi lần thay đổi giá/scope → tạo Quote phiên bản mới (V2, V3) để trace.

🔵 PHASE 4 — SALES ORDER CREATION (sau Closed Won)
KỊCH BẢN 8: Deal Closed Won → tạo Sales Order
Bối cảnh WORKFLOW 8.1 — Thực hiện update Closed Won → Sales Order → Project WORKFLOW 8.2 — Thực hiện update Accepted → Sales Order → Project
Deal đã chốt thành công Trigger: Deal Stage = Closed Won Trigger: Quote Stage = Accepted
Khách đồng ý giá / điều khoản (email confirm / ký hợp đồng / PO) Condition: Quote Stage != Accepted Condition: Deal Stage != Closed Won
Action (auto): Action (auto):
1️⃣ Deal _ 1. Update Quote Stage: Accepted 1. Update Deal Stage = Closed Won
Deal Stage: Closed Won 2. Create Sales Order (link Quote/Deal) 2. Create Sales Order (link Quote/Deal)
Update (Note): Deal đã chốt thành công và tiến tới ký hợp đồng chính thức 3. Create Project (link Deal/Sales Order) 3. Create Project (link Deal/Sales Order)
Close Date set = ngày chốt 4. Create Tasks: 4. Create Tasks:
“Assign coder” (Owner: WD/Manager) “Assign coder” (Owner: WD/Manager)
2️⃣ Quote _ 5. Create Event: 5. Create Event:
Quote Stage: Accepted “Kickoff meeting” (planned) “Kickoff meeting” (planned) 6. Notify Coder/QC/WD 6. Notify Coder/QC/WD
3️⃣ Sales Order

- Auto create Sales Order WORKFLOW 8.3 — Thực hiện create manual Sales Order
- Sales Order Status: Created Trigger: Create manual Sales Order
  Condition:
  4️⃣ Project + Deal Stage != Closed Won
- Auto create Project + Quote Stage != Accepted
- Tạo Project Tasks: Action (auto):
  Assign coder 1. Update Deal Stage = Closed Won
  QC checklist 2. Update Quote Stage = Accepted
  Delivery milestone

KỊCH BẢN 9: Deal Closed Lost (thất bại)
Bối cảnh WORKFLOW 9.1 — Thực hiện update Closed Lost → Stop WORKFLOW 9.2 — Thực hiện update Rejected → Stop
khách chọn vendor khác Trigger: Deal Stage = Closed Lost Trigger: Quote Stage = Rejected
không đủ budget Condition: Quote Stage != Rejected Condition: Deal Stage != Closed Lost
không liên lạc được quá lâu Action (auto): Action (auto): 1. Update Quote Stage = Rejected 1. Update Deal Stage = Closed Lost
1️⃣ Deal _ 2. Set Remind flags (optional keep) 2. Set Remind flags (optional keep)
Deal Stage: Closed Lost 3. Close all open tasks (optional) 3. Close all open tasks (optional)
Update (Note): mô tả lý do 4. Notify manager (optional) 4. Notify manager (optional)
Failure Reason (picklist)
2️⃣ Quote _
Quote Stage: Rejected
3️⃣ Stop reminders

- Tất cả WF2–WF5 phải không chạy nữa (điều kiện đã đề cập)

Bản Pipeline FCJ cho Deal Stage gợi ý

1. Needs Analysis (Estimator)
2. Proposal / Price Quote (O)
3. Follow-up 1 (P)
4. JP Calling (Q/R)
5. Negotiation or Review
6. Follow-up 2 (W)
7. Final Follow-up / Free Campaign (X)
8. Closed Won
9. Closed Lost

Tổng hợp 1 bảng duy nhất (rất đáng lưu)
Module Khi nào dùng
Lead Status Mỗi lần tương tác
Task Luôn luôn phải có
Event Khi có hẹn
Documents Khi có trao đổi tài liệu
Live Chat Khách chủ động
SMS Nhắc nhanh
Deal Khi bắt đầu bán
Quote Khi báo giá
Esign Khi chốt
1️⃣ SOP CHUẨN 1 TRANG CHO SALES (Lead → Cash)
Mục tiêu

- Không để Lead chết
- Không convert sai thời điểm
- Không mất pipeline
- Không bỏ sót follow-up

QUY TRÌNH CHUẨN
BƯỚC 1 — CREATE LEAD

- Nhập đầy đủ: Name, Phone, Email, Source
- Gán Assigned To
- Lead Status: Not Contacted
- Tạo Task: Call trong 24h

BƯỚC 2 — LIÊN HỆ LẦN ĐẦU
Sau khi gọi/email/chat:
Tình huống Lead Status Hành động
Không bắt máy Attempted to Contact Tạo Task gọi lại
Có trao đổi Contacted Ghi Note
Không quan tâm Lost Lead Kết thúc
BƯỚC 3 — CÓ NHU CẦU

- Lead Status: In Progress
- Tạo Event (Meeting)
- Upload Documents nếu có

BƯỚC 4 — SAU MEETING
Nếu có budget & timeline:

- Lead Status: Qualified
- Convert Lead → tạo Deal

BƯỚC 5 — DEAL PROCESS
Stage Hành động
Needs Analysis Soạn proposal
Proposal Tạo Quote
Negotiation Điều chỉnh Quote
Closed Won Tạo Sales Order/Invoice
QUY TẮC VÀNG
✅ Mỗi Lead phải có Task
✅ Không convert nếu chưa đủ điều kiện
✅ Mỗi Deal phải có Stage rõ ràng
✅ Không để Deal đứng yên quá 7 ngày
2️⃣ FLOW DIAGRAM LEAD → CASH
LEAD (CREATED)
↓
Assign Sales
↓
Task: Call / Email
↓
Update Lead Status
↓
Meeting (Event)
↓
IF Qualified
↓
Convert Lead
↓
DEAL (CREATED)
↓
Needs Analysis (Estimating)
↓
Create Quote
↓
Negotiation
↓
Esign Contract
↓
Closed Won
↓
SALES ORDER (CREATED)
↓
Invoice
↓
Payment Received
↓
CASH
3️⃣ THIẾT KẾ WORKFLOW TỰ ĐỘNG ĐỔI STATUS (Vtiger)
Workflow 1 — Sau khi tạo Lead
Trigger: When Lead Created
Action:

- Set Lead Status = Not Contacted
- Create Task: Call within 1 day

Workflow 2 — Khi Event Held
Trigger: When Event Status = Held
Action:

- Update Lead Status → In Progress

Workflow 3 — Khi Deal Stage = Closed Won
Trigger: Deal Stage changes to Closed Won
Action:

- Auto create Sales Order
- Auto create Project (nếu có mapping)

Workflow 4 — Follow-up Reminder
Trigger: If Lead Status = Attempted to Contact
Condition: No activity for 3 days
Action: Send Reminder Email to Sales
4️⃣ MAPPING QUY TRÌNH FCJ / ĐƠN HÀNG NGOÀI
Dựa trên mô hình bạn từng làm (Booking → Assign → QC → Delivery):
📌 PHASE 1: BOOKING
FCJ Step Vtiger Module
Khách gửi yêu cầu Lead
Director nhập sheet Create Lead
Giao cho Sales Assigned To
📌 PHASE 2: ESTIMATE
FCJ Step Vtiger Module
Estimator báo giá Deal
Gửi báo giá Quote
Xin duyệt nội bộ Approval
📌 PHASE 3: CONFIRM ORDER
FCJ Step Vtiger Module
Khách đồng ý Deal Stage → Closed Won
Ký hợp đồng Esign
Tạo đơn hàng Sales Order
📌 PHASE 4: TRIỂN KHAI
FCJ Step Vtiger Module
Assign Coder Project
QC Project Task
Delivery Milestone
📌 PHASE 5: THANH TOÁN
FCJ Step Vtiger Module
Xuất hóa đơn Invoice
Nhận tiền Payment
4️⃣ THIẾT KẾ PIPELINE RIÊNG CHO FCJ
Pipeline mới: “FCJ Order Pipeline”
Stages:

1. Booking Received
2. Estimating
3. Quote Delivered
4. Negotiating
5. Confirmed
6. In Production
7. QC
8. Ready Delivery
9. Delivered
10. Closed

Mapping stage theo thực tế FCJ
Stage Ai chịu trách nhiệm
Booking Received WD
Estimating Estimator
Quote Delivered WD
Confirmed Director
In Production Coder
QC QC team
Ready Delivery WD
Delivered Comtor
Closed Finance
4️⃣ TỐI ƯU AUTOMATION NÂNG CAO (Deal)
Workflow 1 — Khi Stage = Estimating
Auto:

- Create Task: Estimation deadline 2 ngày
- Assign to Estimator

Workflow 2 — Khi Quote Delivered
Auto:

- Send Email template
- Create follow-up task sau 3 ngày

Workflow 3 — Khi Stage = Confirmed
Auto:

- Create Sales Order
- Create Project
- Assign Coder

Workflow 4 — Khi Project Milestone = QC
Auto:

- Assign QC
- Create QC checklist task

Workflow 5 — Khi Invoice Paid
Auto:

- Update Deal → Closed
- Tính Profit tự động
